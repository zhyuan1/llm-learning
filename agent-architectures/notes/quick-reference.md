# Agent Architectures Quick Reference

## 一、先用两个层次看 Agent 架构

### 1. 推理 / 编排层

谁决定下一步怎么做：

- 代码预先写死流程
- 模型动态决定流程
- 一个 agent 拆任务给多个 agent
- 不同 agent 接力完成不同阶段
- 多个 agent 通过事件或消息互相协作

### 2. 运行时 / 基础设施层

系统怎么把任务跑稳：

- 状态放哪里
- 是否能 checkpoint / 恢复
- 是否支持 human-in-the-loop
- 是否有 sandbox 和 guardrails
- 是否能分布式扩展

## 二、主流架构分类

### 1) 增强型单 Agent 循环

**结构**
- 一个 LLM
- 一组工具
- 可选 memory / retrieval
- 循环：思考 → 调工具 → 读结果 → 继续

**优点**
- 简单
- 调试成本低
- 很多任务已经够用

**缺点**
- 宽任务、长任务、并行任务容易吃力

**适用**
- 编码助手
- 轻量客服
- 个人效率工具

### 2) 预定义工作流 / DAG / 状态机

**结构**
- Prompt chaining
- Router
- Parallelization
- Evaluator-optimizer
- Graph / state machine

**优点**
- 可控
- 稳定
- 好测
- 适合审计和合规

**缺点**
- 灵活性不如自由 agent
- 需求变化时要改编排逻辑

**适用**
- 企业流程
- 审批流
- 文档抽取与校验
- 有明确阶段的生产任务

### 3) Orchestrator-Worker 多 Agent

**结构**
- 一个 lead / orchestrator 拆任务
- 多个 worker / subagent 并行执行
- 上层 agent 汇总结果

**优点**
- 适合宽任务与并行探索
- 降低单个 agent 的上下文压力

**缺点**
- token 成本高
- 重复劳动风险高
- 汇总质量决定上限

**适用**
- 深度研究
- 搜索与比对
- 多模块代码理解
- 宽范围情报搜集

### 4) Handoff / 专家接力式多 Agent

**结构**
- 当前 agent 识别边界
- 将任务交给另一个更合适的 agent

**优点**
- 角色边界清晰
- 适合前台接待 → 专家处理

**缺点**
- handoff 设计不好会相互踢皮球

**适用**
- 客服分流
- 销售/技术支持分层
- 通用入口 → 专项 agent

### 5) 图式 / 持久化 / 可恢复 Agent

**结构**
- 显式 state
- checkpoint / persistence
- human-in-the-loop
- 从失败点恢复

**优点**
- 适合长任务
- 稳定性高
- 可运维性强

**缺点**
- 工程复杂度更高

**适用**
- 长时间运行流程
- 需要人工介入
- 企业生产级 agent

### 6) 事件驱动 / 分布式多 Agent

**结构**
- agents as services
- message / event 通信
- 可跨机器、跨语言

**优点**
- 扩展性强
- 适合复杂系统集成

**缺点**
- 最重
- 调试和一致性处理复杂

**适用**
- 大型企业系统
- 多服务、多语言、多团队环境

### 7) 托管服务型 Agent Runtime

**结构**
- Session / state 外置
- Harness / orchestrator 无状态化
- Sandbox 执行环境隔离

**优点**
- 安全边界清晰
- 更适合托管、多租户、产品化

**缺点**
- 基础设施成本高

**适用**
- Agent 平台
- 多租户 SaaS
- 高隔离要求的托管系统

## 三、它们的共同点

几乎所有主流 Agent 架构都共享这几个核心部件：

1. `LLM + Tools + State`
2. 需要明确的工具接口
3. 需要 guardrails
4. 需要 tracing / observability
5. 都在尝试把模型能力和系统能力解耦

## 四、真正的差异在哪里

最关键的差异只有五个维度：

1. **控制权在谁手里**：代码还是模型
2. **任务分解方式**：不分解、静态分解、动态分解、角色分解
3. **状态存储方式**：进程内、checkpoint、外部 session/event log
4. **通信方式**：tool call、handoff、shared state、message bus
5. **复杂度换来的收益**：灵活性、可靠性、扩展性还是安全性

## 五、框架映射

### Anthropic

- `Building Effective AI Agents`：最清楚地区分了 workflow 与 agent
- `multi-agent research system`：典型 orchestrator-worker，多 agent 并行研究

### OpenAI Agents SDK

- 核心 primitives：agent、tools、handoffs、guardrails、traces
- 更适合做从单 agent 到 handoff/manager orchestration 的系统

### LangGraph

- 最鲜明的是 graph + persistence + HITL + durable execution
- 强项是长任务、可恢复、生产态 stateful workflow

### AutoGen

- 强调 event-driven、多 agent 会话、distributed runtime
- 更适合复杂协作和分布式编排

## 六、适用范围速查

### 用单 Agent loop
当任务：
- 不太长
- 不太并行
- 不需要复杂恢复

### 用 workflow / graph
当任务：
- 步骤可预定义
- 需要稳定性
- 需要 checkpoint
- 需要 human-in-the-loop

### 用 orchestrator-worker
当任务：
- 很宽
- 可并行
- 信息量大
- 需要多路探索再汇总

### 用 handoff
当任务：
- 角色边界清晰
- 适合接力
- 不需要大规模并行

### 用 event-driven / distributed runtime
当任务：
- 本质上是分布式系统问题
- 需要跨服务、跨语言、跨机器协作

### 用 managed runtime
当任务：
- 要做平台
- 要做多租户
- 要求执行隔离和高安全边界

## 七、最重要的一条经验

**先做最简单能跑通的那个。**

很多场景里，优化单次 LLM 调用，加上 retrieval / tools，就已经足够；不是所有问题都值得上 multi-agent。