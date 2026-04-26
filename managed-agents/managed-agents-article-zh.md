# 托管智能体服务架构：Anthropic 如何把大脑和双手分开

**状态：** 草稿（Phase 4 完成，Phase 5 精修）  
**日期：** 2026-04-12  
**来源：** Anthropic Engineering 博客、掘金技术社区、搜狐技术报道

---

## 1. 问题：宠物式架构撑不住

早期的智能体系统把所有东西塞进一个容器：LLM 推理、编排循环、代码执行、文件操作、会话状态。这个容器是一只"宠物"——坏了要抢救，不能随便换，调试时甚至需要接触用户数据。

具体问题：
- 容器故障 = 整个会话永久丢失
- 调试只能靠 WebSocket 事件流，无法定位故障源
- 客户的资源在自己的 VPC 里，唯一的接入方式是网络对等连接
- 每个智能体实例都要独占容器，资源浪费严重

更深层的问题：**Harness 编码了关于模型能力的假设，而这些假设会随着模型进步而过时。** 当所有组件都耦合在一起时，升级任何一个都会牵动其余所有部分。

---

## 2. 核心思路：虚拟化智能体的组件

Anthropic 的解法借鉴了操作系统设计。操作系统把硬件抽象成进程和文件，硬件怎么换都不影响上层接口。同样，Managed Agents 把智能体拆成三个可独立替换的组件，通过稳定的接口连接：

| 组件 | 角色 | 关键特性 |
|------|------|---------|
| **Session（会话）** | 持久化记忆 | 追加写入的事件日志，存储在容器外部 |
| **Harness（编排器）** | 大脑的控制循环 | 无状态，调用 Claude 并路由工具输出 |
| **Sandbox（沙箱）** | 双手的执行环境 | 隔离的代码执行容器，按需创建 |

设计原则：**接口比实现更持久。** 只要 Session、Harness、Sandbox 之间的接口契约不变，每一层的实现都可以独立演进。

---

## 3. Session：追加写入的事件日志

Session 是所有事件的持久化记录，存储在容器外部（Postgres、SQLite 等）。它不是上下文窗口——上下文窗口有长度限制且随对话消耗，Session 则是完整的外部记忆。

### 写入

Harness 通过 `emitEvent(id, event)` 向 Session 写入事件。每一次工具调用、每一条 Claude 的回复、每一个错误都被记录。事件流只追加，不修改。

### 读取

Harness 通过 `getEvents()` 接口从 Session 读取事件。这个接口支持灵活的检索模式：
- 从上次读取位置继续
- 回溯到某个特定时刻之前
- 重新读取先前的上下文
- 按时间片段切片

这意味着 Harness 可以自行决定如何组织上下文——包括为了提高 prompt cache 命中率而重新排列事件。这种灵活性很关键，因为未来的模型可能需要完全不同的上下文工程策略。

### 为什么不直接压缩上下文窗口

压缩是不可逆的。一旦你把 10 条事件摘要成 1 条，原始细节就丢了。Session 把完整的事件流保留在外部，让 Harness 按需选取，不做不可逆的决定。

---

## 4. Harness：无状态的编排器

Harness 是调用 Claude 的控制循环。它读取 Session 中的事件，构造 prompt 发给 Claude，接收 Claude 的输出，路由工具调用到 Sandbox，然后把结果写回 Session。

关键设计：**Harness 不保存任何状态。** 所有状态都在 Session 里。

### 故障恢复

Harness 崩溃时，启动一个新的 Harness，执行：
1. `wake(sessionId)` 唤醒会话
2. `getSession(id)` 获取事件日志
3. 从最后一条事件继续执行

这就是"牲口而非宠物"——Harness 是可替换的基础设施。崩了就换一个新的，用户无感知。

### 工具调用契约

Harness 通过一个极简接口调用 Sandbox：

```
execute(name, input) → string
```

`name` 是工具名，`input` 是输入字符串，返回结果字符串。Harness 不关心 Sandbox 是一个容器、一台手机还是一个 Pokemon 模拟器。任何能接受名称和输入、返回字符串的东西都可以是 Sandbox。

如果工具调用失败，Harness 把失败包装成工具调用错误返回给 Claude，Claude 可以决定重试或换一种方式处理。

---

## 5. Sandbox：按需创建的执行环境

Sandbox 是代码执行和文件操作的隔离环境。容器通过 `provision({resources})` 按需创建，只在工具调用时才需要。

### 隔离技术

业界有两种主流方案：

**Firecracker microVM**：硬件级虚拟化，独立的客户操作系统内核。隔离强度最高（5/5），冷启动 125-150ms。AWS Lambda 和 Manus 使用这种方案。

**gVisor 容器**：用户空间实现的轻量级内核，Sentry 运行时拦截系统调用。隔离强度 4/5（需要额外加固），冷启动 <50ms。

### 多租户隔离

核心原则：**严禁多用户共用实例。**

违反这条原则会导致：用户可以通过 `ls -R /` 看到其他用户的数据，一个用户的死循环可以耗尽其他用户的资源，审计日志被污染。

隔离方式：
- 每个用户/任务分配独立的沙箱实例
- 控制面注入用户级别的配置前缀（如 `memory_prefix: f'user-{user_id}/assistant-{assistant_id}/'`）
- 存储和内存层面的命名空间隔离

---

## 6. 安全边界：凭证永远不进沙箱

这是架构中最重要的安全决策之一。Claude 生成的代码在沙箱里执行，而沙箱可能被 prompt injection 攻击利用。如果凭证在沙箱里，攻击者就能拿到它们。

Anthropic 的方案：

### Git 认证
仓库的访问令牌在初始化阶段用于克隆仓库，并写入本地 git remote 配置。之后 `push` 和 `pull` 操作通过本地 remote 完成，智能体本身不接触令牌。

### OAuth 认证
OAuth 令牌存储在安全的 vault 中。Claude 通过 MCP 协议调用外部工具时，请求经过一个专用代理（proxy）。代理拿着与会话关联的 token 去 vault 取真实凭证，然后代为调用外部服务。Claude 全程不碰 OAuth 令牌。

### Tool Proxy 模式（通用方案）
1. 沙箱内的工具只持有临时的 `sandbox_token`（短期 JWT 或 Redis key）
2. 控制面代理验证 token 后，用真实的 API key 调用外部 API
3. 即使容器被攻破，API key 也不会泄露

---

## 7. 性能收益

解耦带来了显著的延迟改善，原因很直接：之前要等容器启动完毕才能开始推理，现在 Harness 拿到 Session ID 就能立即开始推理，沙箱在后台按需创建。

| 指标 | 改善幅度 |
|------|---------|
| p50 首 token 延迟 | 下降约 60% |
| p95 首 token 延迟 | 下降超过 90% |

容器按需创建还意味着不需要为每个智能体实例预留容器。Harness 无状态化后可以水平扩展，多个 Harness 实例可以处理同一个 Session。

---

## 8. Many Brains, Many Hands

解耦之后，系统获得了组合的灵活性：

- **一个大脑连接多双手**：一个 Harness 可以同时调用多个沙箱（不同语言的运行时、不同的外部服务），它们之间没有耦合。
- **大脑之间互相委派**：多个 Harness 可以协作。一个 Harness 可以把子任务交给另一个 Harness，后者用自己的 Sandbox 执行。共享 Session 使并行任务处理成为可能。
- **手可以在大脑之间传递**：没有哪双手被绑定在特定的大脑上。

Harness 不关心 Sandbox 是容器、手机还是其他系统。这种不可知性（agnostic）是系统可扩展性的来源——不需要改编排层就能接入新的执行环境。

---

## 9. 核心心智模型

Managed Agents 架构的核心是一个操作系统级别的抽象：把智能体的组件虚拟化，用稳定的接口隔离它们。

三个组件各自独立：
- **Session**：追加写入的事件日志，是持久化的外部记忆，比上下文窗口更持久
- **Harness**：无状态的编排循环，崩溃可恢复，可水平扩展
- **Sandbox**：按需创建的隔离执行环境，凭证永远留在外面

一个极简的工具调用契约把它们串起来：`execute(name, input) → string`。

这个架构让每一层都可以独立演进。模型变强了，Harness 的假设可以更新而不动 Sandbox。需要新的执行环境，加一个新的 Sandbox 实现而不动 Harness。需要不同的上下文策略，改 `getEvents()` 的调用方式而不动 Session 的存储格式。

接口比实现更持久。这是整个设计最重要的一句话。

---

## 参考来源

- [Anthropic Engineering: Managed Agents](https://www.anthropic.com/engineering/managed-agents) — 一手资料
- [掘金：Agent Sandbox 架构深度解析](https://article.juejin.cn/post/7618805017437650987) — 沙箱隔离技术对比
- [搜狐：Anthropic 的 Harness 哲学](https://www.sohu.com/a/1008149966_122189055) — 中文技术报道
- [CNBlogs：全面解读 Managed Agents](https://www.cnblogs.com/wind-xwj/articles/19843533) — 中文技术解读
