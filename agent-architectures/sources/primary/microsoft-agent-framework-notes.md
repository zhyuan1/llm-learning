# Microsoft Agent Framework

- URL: https://github.com/microsoft/agent-framework
- 类型: 官方仓库 README / 官方框架说明
- 记录日期: 2026-04-30

## 这份资料提供了什么

Microsoft Agent Framework 很值得关注，因为它像是 AutoGen 思路的工程化演进版，更偏企业生产系统。

## 核心架构特征

### Graph-based workflows
- 用图来连接 agent 和 deterministic functions
- 支持 data flow、streaming、checkpointing

### Enterprise runtime concerns
- OpenTelemetry observability
- middleware
- DevUI
- hosting / durable workflows
- human-in-the-loop

### Multi-language support
- Python + .NET
- 更强调跨语言和企业落地

## 它和 AutoGen 的关系

- 官方文档里明确提供从 AutoGen 迁移的指引
- 可以理解为：从多 agent conversation 走向更完整的 workflow/runtime/framework

## 适用范围判断

### 适合
- 企业级 agent orchestration
- 图式工作流
- 强 observability、强部署要求的系统
- 需要多语言支持的团队

### 不太适合
- 快速轻量验证
- 极简单 agent 场景

## 对本次研究的贡献

这份资料说明了另一个趋势：

> Agent 架构会逐步从“对话式实验框架”演进为“带工作流、观测、持久化、部署能力的工程框架”。