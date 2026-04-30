---
article: ../article-draft.md
type: mixed
density: per-section
style: diagram-dark
palette: default
image_count: 5
update_mode: update
backend: baoyu-diagram
---

## Illustration 1
**Position**: 引言后，第一节前
**Purpose**: 先给读者一张总览图，把文章的两条分类主轴一次讲清楚
**Visual Content**: 二维矩阵，横轴为“控制权：代码主导 → 模型主导”，纵轴为“系统重心：提示词编排 → 运行时能力”；在图中标出增强型单 Agent、workflow/graph/flows、manager+tools、handoff、orchestrator-worker、conversation-based multi-agent、event-driven runtime、managed/runtime-heavy systems
**Filename**: 01-framework-agent-matrix.svg

## Illustration 2
**Position**: 第三节《从“循环”到“流程”》后
**Purpose**: 让读者看到系统是如何从单循环升级到显式编排的
**Visual Content**: 演进流程图：单 Agent loop → prompt chaining / routing / parallelization / evaluator-optimizer → graph state / checkpoint / interrupt → 生产级流程系统
**Filename**: 02-flowchart-orchestration-evolution.svg

## Illustration 3
**Position**: 第四节《多 Agent 的第一条主线》后
**Purpose**: 用并排对比图区分 manager、handoff、orchestrator-worker 三种多 Agent 形态
**Visual Content**: 三栏对比图，分别展示控制权位置、用户面对谁、典型优势、典型失效方式
**Filename**: 03-comparison-multi-agent-patterns.svg

## Illustration 4
**Position**: 第五节《Conversation-based 到 Event-driven Runtime》后
**Purpose**: 解释多 Agent 从“对话协作”走向“系统工程”的演进轨迹
**Visual Content**: 从左到右的演进图：conversation-based coordination → message bus / state sync → graph workflow + checkpoint + observability → enterprise runtime
**Filename**: 04-timeline-conversation-to-runtime.svg

## Illustration 5
**Position**: 第六节《运行时能力开始成为主角》后
**Purpose**: 把生产级 Agent 系统真正需要的运行时能力打包成一张结构图
**Visual Content**: 分层框架图，展示 inference、state & recovery、tool isolation、human-in-the-loop、trace/metrics、deployment/runtime 等能力，以及它们对应的常见故障点
**Filename**: 05-framework-runtime-capability-stack.svg
