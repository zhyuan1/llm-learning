# LangGraph

- URL: https://github.com/langchain-ai/langgraph
- 类型: 官方仓库 README / 官方文档入口
- 记录日期: 2026-04-30

## 这份资料提供了什么

LangGraph 代表的是另一条非常重要的路线：

> 不把 agent 看成一个简单循环，而是把它建模为一个带状态、可恢复、可中断的 graph/state machine。

## 核心特征

### Graph / state-machine abstraction
- 节点和边组成图式工作流
- 支持 branching 和 subgraphs

### Persistence / durability
- durable execution
- 自动从失败点恢复
- 支持长时运行

### Memory
- 短期 memory
- 长期 persistent memory

### Human-in-the-loop
- 支持 execution interrupt
- 允许中途检查和修改状态

## 它和简单 agent loop 的差异

- 不是只靠“模型下一步决定做什么”
- 更强调显式 state 和执行恢复
- 更适合长流程和生产系统
- 更容易插入人工审批和校验点

## 适用范围判断

### 适合
- 长时运行任务
- 需要 checkpoint 的流程
- 需要 human approval / HITL
- 企业内部稳定工作流

### 不一定最适合
- 很轻量、一次性的小任务
- 仅需简单工具循环的 agent

## 对本次研究的贡献

LangGraph 让本次分类更完整：

> Agent 架构不只是“几个 agent 怎么协作”，还包括“状态是否显式、是否持久、是否可恢复”。