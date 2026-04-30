# CrewAI — Flows

- URL: https://docs.crewai.com/en/concepts/flows
- 类型: 官方文档
- 记录日期: 2026-04-30

## 这份资料提供了什么

CrewAI 很有代表性，因为它明确区分了两层：

- **Crews**：执行任务的 agent 团队
- **Flows**：编排和控制整个工作流的上层机制

## 核心架构含义

### Flows 是什么
- 一个 event-driven orchestration layer
- 支持 start / listen / router 等执行原语
- 可以做条件路由、分支、循环和并行启动

### Flows 和 Crews 的关系
- Flows 负责 orchestration
- Crews 负责具体任务执行
- 一个 Flow 可以组合多个 Crews

### 状态模型
- 每个 Flow 有唯一实例 ID
- 支持结构化或非结构化 state
- 支持 persistence

## 它更像哪种架构

CrewAI 的这套设计说明：

> 它并不只是“多 agent 团队”，而是“角色团队 + 工作流编排”双层结构。

这更接近：
- 工作流 / DAG / 状态机
- 在某些场景下再叠加 agent 团队执行

## 适用范围判断

### 适合
- 多步骤企业自动化
- 有显式阶段和条件分支的任务
- 既要 agent，又要精确工作流控制的场景

### 不太适合
- 极简轻量原型
- 不需要显式编排的短任务

## 对本次研究的贡献

CrewAI 强化了一个判断：

> 很多所谓“多 agent 架构”，本质上是“workflow orchestration + agent execution”的组合。