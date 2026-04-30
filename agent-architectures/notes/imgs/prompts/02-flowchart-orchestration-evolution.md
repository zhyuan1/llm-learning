---
type: flowchart
style: diagram-dark
palette: default
aspect: 16:9
language: zh
output: ../02-flowchart-orchestration-evolution.svg
---

# 从单 Agent 循环到流程编排的演进图

## Goal
让读者看到系统复杂度增长时，第一步升级通常不是更多 Agent，而是更强的编排能力。

## Layout
- 左到右流程图
- 5 个主要阶段，中间用箭头连接

## Steps
1. 单 Agent loop
   - 关键词：LLM、tools、memory、retrieval
2. Workflow patterns
   - 关键词：prompt chaining、routing、parallelization、evaluator-optimizer
3. Graph orchestration
   - 关键词：state、edges、nodes
4. Recoverable execution
   - 关键词：checkpoint、interrupt、human-in-the-loop
5. Production workflow system
   - 关键词：可恢复、可追踪、可审批、可审计

## Labels
- 标题：从“循环”到“流程”的升级路径
- 底部注释：路径越往右，灵活性通常下降，但可控性和恢复能力上升

## Style Notes
- 不要画成过度复杂的 BPMN
- 强调“升级的是编排能力，不一定是 agent 数量”
- 每个阶段用一个清晰的盒子，内部 2 到 4 个关键词即可
