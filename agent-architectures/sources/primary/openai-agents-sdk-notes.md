# OpenAI Agents SDK

- URL: https://openai.github.io/openai-agents-python/
- 类型: 官方文档
- 记录日期: 2026-04-30

## 这份资料提供了什么

OpenAI Agents SDK 不是在给一种固定架构，而是在给一组最小但够用的 primitives：

- agent
- tools
- handoffs
- guardrails
- traces

## 核心抽象

### Agent
- 本质上是“带 instructions 和 tools 的 LLM”
- 内建 agent loop

### Tools
- function tools
- hosted tools
- agents as tools
- MCP server tools

### Handoffs
- agent 间任务交接
- 支持 handoff 风格和 manager 风格的编排

### Guardrails
- 输入/输出验证
- 快速失败

### Traces
- 可视化、调试、监控
- 更适合生产使用

## 架构倾向

这套 primitives 最适合：
- 单 agent 到多 agent 的渐进式演化
- 专家接力式多 agent
- 需要 guardrails 和 trace 的生产系统

它不强制要求 graph，也不强制要求 event bus。

## 适用范围判断

### 适合
- 从简单到中等复杂度的 agent 系统
- 需要 handoff / manager orchestration
- 需要良好 observability 的应用

### 不一定最适合
- 显式图状态机
- 特别长时、强持久化工作流
- 重分布式 runtime 场景

## 对本次研究的贡献

OpenAI Agents SDK 帮助明确了另一个重要视角：

> 有些系统不是靠“宏大架构”取胜，而是靠一组足够稳的 primitives 搭出来。