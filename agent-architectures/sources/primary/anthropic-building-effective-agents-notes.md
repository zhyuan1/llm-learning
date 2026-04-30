# Anthropic — Building Effective AI Agents

- URL: https://www.anthropic.com/research/building-effective-agents
- 类型: 官方研究/实践文章
- 记录日期: 2026-04-30

## 这份资料提供了什么

这篇文章给出了一个非常实用的官方 taxonomy：

> workflow 是通过预定义代码路径编排 LLM 和工具；agent 则让 LLM 动态决定如何使用工具和完成任务。

## 核心架构分类

### Foundation: Augmented LLM
- 检索
- 工具调用
- memory

这是所有更复杂系统的基本单元。

### Workflow patterns
1. Prompt chaining
2. Routing
3. Parallelization
4. Orchestrator-workers
5. Evaluator-optimizer

### Agents
- LLM 在循环中基于环境反馈自主决定下一步
- 需要停止条件与 guardrails
- 成本和错误传播风险高于 workflow

## 最关键的判断

Anthropic 的立场非常明确：

- 先从最简单的方案开始
- 只有在复杂度带来明确收益时，才引入 agentic behavior
- 很多应用里，优化单次 LLM 调用 + retrieval + 少量工具，已经足够

## 适用范围判断

### 适合 workflow
- 步骤已知
- 子任务边界明确
- 更看重稳定性和可控性

### 适合 agent
- 问题开放
- 不能预定义完整路径
- 模型需要在多轮中动态决策

## 对本次研究的贡献

这份资料是本次分类的理论起点，尤其是：
- workflow vs. agent 的边界
- 何时不需要 multi-agent
- 为什么不能把“用了工具”直接等同于“用了 agent”