# OpenAI Agents SDK — Agent Orchestration

- URL: https://openai.github.io/openai-agents-python/multi_agent/
- 类型: 官方文档
- 记录日期: 2026-04-30

## 这份资料提供了什么

这页文档把多 agent 编排明确拆成两大类：

1. **LLM-based orchestration**
2. **Code-based orchestration**

这对架构分析很重要，因为它说明“多 agent”并不等于“全都交给模型自己决定”。

## 核心模式

### Agents as Tools
- 一个 manager agent 持续掌控全局
- specialist agent 只是被调用来完成受限子任务
- 最终答案仍由 manager 输出

### Handoffs
- 当前 agent 负责分流
- 被选中的 specialist 接管后续交互
- 更像角色接力而不是内部工具调用

### Code-based patterns
- sequential chaining
- feedback loops
- parallel execution
- structured outputs for routing

## 关键判断

### 用 agents as tools
当你需要：
- 一个 agent 保持用户面的一致性
- specialist 只辅助，不接管
- 统一 guardrails 和输出口径

### 用 handoff
当你需要：
- specialist 直接接管当前回合
- prompt 和角色边界更聚焦
- 路由本身就是业务流程的一部分

### 用 code-based orchestration
当你更在意：
- 可预测性
- 成本
- 时延
- 稳定性

## 对本次研究的贡献

这份资料把“多 agent”内部再细分出了两种完全不同的控制模型：

- manager 保持控制
- specialist 直接接管

也进一步证明：**架构选择核心在控制权，而不只是 agent 数量。**