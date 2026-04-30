---
type: framework
style: diagram-dark
palette: default
aspect: 16:10
language: zh
output: ../01-framework-agent-matrix.svg
---

# Agent 架构二维分类矩阵

## Goal
画一张可直接嵌入技术文章的二维框架图，帮助读者理解 Agent 架构不是单 Agent / 多 Agent 的单轴分类，而是两个维度共同决定。

## Layout
- 使用二维矩阵布局
- 横轴：控制权，左侧“代码主导”，右侧“模型主导”
- 纵轴：系统重心，下侧“提示词编排”，上侧“运行时能力”
- 中央区域标出“混合式”

## Zones
- 左下：workflow / DAG / graph / flows
- 右下：增强型单 Agent loop、autonomous agent
- 中下偏中：manager + agents as tools、handoff、orchestrator-worker
- 上半区：conversation-based multi-agent、event-driven runtime、managed/runtime-heavy systems

## Labels
- 标题：Agent 主流架构的二维分类
- 副标题：控制权 × 系统重心，比“单 Agent / 多 Agent”更能解释差异
- 点位标签：
  - 增强型单 Agent
  - Workflow / Graph / Flows
  - Manager + tools
  - Handoff
  - Orchestrator-worker
  - Conversation-based
  - Event-driven runtime
  - Managed / runtime-heavy systems
- 角标说明：
  - 左侧更稳、更易审计
  - 右侧更灵活、更不确定
  - 下侧偏编排
  - 上侧偏运行时

## Style Notes
- 面向工程长文，信息优先，不要做装饰性插画
- 保持标签清晰，允许用箭头或半透明区域表现过渡关系
- 用深色技术图风格，保证中文可读
