---
type: timeline
style: diagram-dark
palette: default
aspect: 16:9
language: zh
output: ../04-timeline-conversation-to-runtime.svg
---

# 从对话协作到运行时系统的演进图

## Goal
解释为什么 conversation-based multi-agent 往前走，会逐渐从提示词工程问题变成系统工程问题。

## Layout
- 水平时间线 / 演进线
- 4 个阶段节点，自左向右增强工程性

## Stages
1. Conversation-based coordination
   - 关键词：multi-agent dialogue、角色协商、互相纠错
2. Message & state concerns
   - 关键词：消息传递、状态同步、角色增多
3. Runtime features emerge
   - 关键词：graph workflow、checkpoint、observability、middleware
4. Enterprise runtime
   - 关键词：hosting、多语言、deployment、DevUI、可观测

## Labels
- 标题：多 Agent 为什么会从“对话设计”走向“运行时设计”
- 底部说明：前半段主要贵在推理，后半段主要贵在系统

## Style Notes
- 不是历史时间线，而是能力演进图
- 每个阶段用简短标签，不要长段文字
