# Deep Research Digest

## 1. 这轮深度研究后，主流 Agent 架构可以更细地分成三层

### A. 决策层：谁决定下一步

1. **Code-first**
   - 路径主要由代码定义
   - 代表：workflow / DAG / graph / flows
2. **Model-first**
   - 路径主要由模型根据环境反馈决定
   - 代表：单 agent loop、autonomous agent
3. **Hybrid**
   - 代码定义边界，模型在边界内动态决策
   - 代表：LangGraph、OpenAI Agents SDK 的 manager/tool 模式、很多企业 agent 系统

## 2. 协作层：多个 agent 到底怎么合作

### 1) Tool-style collaboration
一个主 agent 把其他 agent 当工具调用。

- 控制权最集中
- specialist 不接管会话
- 代表：OpenAI `agents as tools`

### 2) Handoff-style collaboration
一个 triage/entry agent 把任务交给 specialist。

- 角色边界最清晰
- specialist 接管当前交互
- 代表：OpenAI `handoff`

### 3) Orchestrator-worker collaboration
上层 orchestrator 拆任务，下层 worker 并行执行。

- 适合宽任务
- 代表：Anthropic multi-agent research system

### 4) Conversation-as-coordination
多个 agent 通过对话本身完成协调。

- 协调协议更自然，但也更松散
- 代表：AutoGen paper

### 5) Event-driven coordination
多个 agent 通过 runtime、消息和事件协作。

- 更接近分布式系统
- 代表：AutoGen runtime、Microsoft Agent Framework、CrewAI Flows 的部分场景

## 3. 运行时层：系统是否把 agent 当成“应用”而不是“prompt”

这是深度研究后最清晰的发现之一：

> 越往生产走，架构重点越从 prompt 编排转向 runtime 设计。

关键 runtime 议题包括：

- state / session 外置
- checkpoint 与 durable execution
- human-in-the-loop
- sandbox / credential isolation
- observability / tracing
- hosting / deployment / middleware

## 4. 几条清晰的演进路线

### 路线 1：单 agent → workflow → stateful graph
- 起点：单 agent loop
- 中间：静态编排
- 生产化：显式状态、checkpoint、恢复、人类介入
- 代表：LangGraph

### 路线 2：单 agent → specialist routing → handoff / manager orchestration
- 起点：单 agent
- 中间：引入 specialist
- 分叉：
  - manager 持续掌控
  - specialist 接管会话
- 代表：OpenAI Agents SDK

### 路线 3：conversation-based multi-agent → event-driven distributed runtime
- 起点：AutoGen 论文的多 agent 对话
- 演进：更强调 runtime、事件、部署、多语言
- 代表：AutoGen → Microsoft Agent Framework

### 路线 4：workflow orchestration + agent execution 双层化
- 上层：显式流程控制
- 下层：crews / agents 执行任务
- 代表：CrewAI Flows + Crews

## 5. 关键分歧其实不是“单 agent 还是多 agent”

深度研究后，更关键的分歧是下面这些：

### 分歧 A：控制权是集中还是分散
- 集中：manager / workflow / graph
- 分散：conversation-based multi-agent

### 分歧 B：协作协议是显式还是隐式
- 显式：graph edge、router、event、handoff
- 隐式：自然语言协商

### 分歧 C：状态是一等公民还是附属物
- 附属物：轻量 loop
- 一等公民：LangGraph / managed runtime / agent frameworks

### 分歧 D：目标是灵活性还是可运营性
- 灵活性优先：自由 agent、多 agent conversation
- 可运营性优先：workflow、graph、runtime-heavy frameworks

## 6. 每类架构的真正适用边界

### 单 agent loop
最适合：
- 任务短
- 目标清晰
- 工具少而稳定

失效边界：
- 问题过宽
- 需要多路探索
- 需要强恢复能力

### Workflow / Graph / Flows
最适合：
- 阶段明确
- 想把系统做稳
- 需要人类审批和恢复

失效边界：
- 新奇度极高、很难预定义结构的任务

### Handoff
最适合：
- 角色边界清楚
- 用户面角色切换合理

失效边界：
- specialist 之间互相依赖很多

### Orchestrator-worker
最适合：
- 可并行搜索
- 可并行阅读
- 多路信息探索再汇总

失效边界：
- 强共享上下文任务
- 严格串行依赖任务

### Conversation-based multi-agent
最适合：
- 多角色协作本身就是问题求解方式的一部分
- 研究和实验性场景

失效边界：
- 需要高稳定、高可审计的生产场景

### Event-driven / distributed runtime
最适合：
- 真正的企业系统集成
- 多服务多语言协作
- 强 deployment / monitoring / runtime concerns

失效边界：
- 小团队、小项目、验证期原型

## 7. 对“主流架构”的最终归纳

如果只保留最核心的一句话：

> 主流 Agent 架构不是一棵树，而是两条主轴交叉形成的矩阵：
> 1. 决策控制权在代码还是模型
> 2. 系统重心在 prompt 编排还是 runtime 能力

所以现实里的框架和系统，大多是这两个维度上的不同折中，而不是某一种纯粹形态。

## 8. 下一步最值得做什么

基于当前资料池，后续可以继续推进三件事：

1. **Digest 细化**：把每类架构再拆成失败模式、成本结构、可观测性要求
2. **Outline**：整理成一篇系统性文章提纲
3. **案例映射**：把 Claude Code、OpenAI Agents SDK、LangGraph、AutoGen、CrewAI、Managed Agents 放到同一张架构地图上