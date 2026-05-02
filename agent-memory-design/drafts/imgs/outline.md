---
type: mixed
style: minimal-flat
palette: default-soft
image_count: 5
language: zh
article: ../agent-memory-design-first-draft.md
---

## Illustration 1

**Position**: 开头，MemGPT 矛盾段落之后
**Purpose**: 帮读者快速理解全文核心矛盾：有限 context window 与长期任务连续性之间的落差。
**Visual Content**: 左侧是有限窗口，右侧是长程任务流，中间有 memory system 作为分层调度器。
**Filename**: 01-framework-context-memory-gap.png

## Illustration 2

**Position**: “Agent 记忆设计要先拆成四个问题”章节开头之后
**Purpose**: 把 What / When / Where / How 四个问题做成总框架，作为后文理解 OpenClaw、Hermes 的地图。
**Visual Content**: 四象限或四段流水线，展示记什么、什么时候记、存哪里、如何维护。
**Filename**: 02-framework-four-questions.png

## Illustration 3

**Position**: OpenClaw 章节，Active Memory 描述之后
**Purpose**: 展示 OpenClaw 的主动召回链路和长期记忆晋升链路。
**Visual Content**: 用户消息进入后，Active Memory 先运行，再注入主回复；同时 daily notes、memory flush、dreaming、MEMORY.md
构成晋升链路。
**Filename**: 03-flowchart-openclaw-memory.png

## Illustration 4

**Position**: Hermes 章节，frozen snapshot 与 session search 描述之后
**Purpose**: 展示 Hermes 的小型常驻记忆、冻结快照、会话检索和压缩/缓存之间的关系。
**Visual Content**: MEMORY.md / USER.md 注入 system prompt frozen snapshot；session_search 走旁路；context compression 与
prompt caching 稳定前缀。
**Filename**: 04-framework-hermes-memory.png

## Illustration 5

**Position**: “再往前看，Agent 记忆正在变得更结构化”章节之后
**Purpose**: 总结前沿趋势，让读者看到 Agent memory 的演进方向。
**Visual Content**: 五条趋势并列：A-Mem 动态连接、MemoryOS 三层分层、Graphiti 时间图谱、Mem0 多信号检索、MIRIX 多模态多
agent。
**Filename**: 05-infographic-frontier-trends.png
