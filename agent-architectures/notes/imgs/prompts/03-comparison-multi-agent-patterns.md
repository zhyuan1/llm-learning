---
type: comparison
style: diagram-dark
palette: default
aspect: 16:10
language: zh
output: ../03-comparison-multi-agent-patterns.svg
---

# 三种多 Agent 协作模式对比图

## Goal
区分 manager、handoff、orchestrator-worker 三种常见多 Agent 协作形态，不要让读者把它们混成一个概念。

## Layout
- 三栏并排比较
- 每栏顶部放模式名，中部画简化交互结构，底部放优势和失败点

## Columns
### 1. Manager + tools
- 结构：用户 → manager → specialist tool 1 / tool 2 / tool 3
- 优势：控制权集中、口径统一、guardrails 易收拢
- 失效点：specialist 被用成昂贵内部工具

### 2. Handoff
- 结构：用户 → router agent → specialist A / specialist B
- 优势：角色边界清楚、prompt 更聚焦
- 失效点：误分流、来回转交

### 3. Orchestrator-worker
- 结构：lead agent → worker A / worker B / worker C → synthesis
- 优势：适合多路探索、并行搜索、并行阅读
- 失效点：拆分重叠、汇总失真、慢 worker 拖全局

## Labels
- 标题：Multi-agent 不是一种形态，而是三种常见协作方式
- 对比维度标签：控制权、用户面对谁、最适合的任务、常见失败方式

## Style Notes
- 强调结构差异，不要堆过多文字
- 每栏最多 4 个 bullet 级信息
