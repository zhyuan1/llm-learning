# Microsoft AutoGen

- URL: https://microsoft.github.io/autogen/stable/index.html
- 类型: 官方文档
- 记录日期: 2026-04-30

## 这份资料提供了什么

AutoGen 代表的不是单纯的“多 agent 对话”，而是更靠近：

- event-driven programming
- conversation-based multi-agent application
- distributed runtime

## 核心抽象

### 分层结构
- Studio：无代码原型
- AgentChat：对话式 agent 开发层
- Core：更底层的 event-driven primitives

### 通信方式
- agent-to-user
- agent-to-agent
- 基于消息/事件的协作

### 分布式能力
- 支持分布式 runtime
- 支持多机器、多语言 agent
- 更适合复杂系统集成

## 它和其他架构的差异

- 比单 agent loop 更强调通信模型
- 比 handoff 更强调 runtime 和分布式协作
- 比 graph workflow 更偏多 agent 系统工程

## 适用范围判断

### 适合
- 多 agent 协同研究
- 复杂业务自动化
- 跨服务、跨语言的 agent 系统

### 不一定最适合
- 轻量场景
- 只需简单 agent loop 的场景
- 小团队快速验证阶段

## 对本次研究的贡献

AutoGen 让本次研究看到了另一条路线：

> 当 agent 数量、语言数量、运行节点数量一起上升时，问题会从“prompt 编排”演化成“分布式系统设计”。