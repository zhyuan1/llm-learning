# AutoGen Paper — Enabling Next-Gen LLM Applications via Multi-Agent Conversation

- URL: https://arxiv.org/abs/2308.08155
- 类型: 原始论文
- 记录日期: 2026-04-30

## 这份资料提供了什么

这篇论文奠定了 AutoGen 这一派的核心思想：

> 通过多个 agent 的对话来构建复杂 LLM 应用，而不是只靠单 agent 顺序调工具。

## 核心架构思想

### Multi-agent conversation
- conversation 本身就是架构机制
- agent 之间通过对话完成协作、分工和纠错

### Agent customization
- agent 可以结合 LLM、human input、external tools
- 可以按不同角色和能力定制

### Communication-driven problem solving
- 不再只是 Agent → Tool → Agent 的单线流程
- 而是 Agent ↔ Agent ↔ Agent 的协作网络

## 它和单 Agent loop 的差异

### 单 Agent loop
- 决策中心单一
- 工具调用线性展开
- 问题分解主要由一个 agent 完成

### AutoGen 式多 agent
- 决策分散到多个 agent
- 对话成为协作协议
- 可以模拟人类团队式问题求解

## 适用范围判断

### 适合
- 需要多角色讨论的问题
- 复杂推理、协同研究、编码协作
- 单 agent 容易卡住的复杂问题

### 不太适合
- 简单直接的任务
- 对成本和时延特别敏感的场景

## 对本次研究的贡献

这篇论文说明：

> 多 agent 架构的一条主线，并不是 workflow graph，而是“conversation as coordination mechanism”。