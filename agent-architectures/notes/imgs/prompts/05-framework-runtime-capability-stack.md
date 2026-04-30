---
type: framework
style: diagram-dark
palette: default
aspect: 16:10
language: zh
output: ../05-framework-runtime-capability-stack.svg
---

# 生产级 Agent 系统运行时能力栈

## Goal
把“运行时能力开始成为主角”这节的核心内容压缩成一张结构图，让读者看到真正的生产问题集中在哪些层。

## Layout
- 自下而上的分层框架图
- 右侧增加“常见故障点”注释区

## Layers
1. Inference
   - LLM、tools、context
2. State & Recovery
   - state store、checkpoint、resume
3. Execution Boundary
   - sandbox、credential isolation、tool proxy
4. Human Oversight
   - approval、interrupt、review
5. Observability
   - trace、metrics、logs、replay
6. Runtime & Deployment
   - scheduler、hosting、multi-service integration

## Failure Notes
- 恢复点不一致
- 工具代理配置不当
- trace 链断掉
- 人工审批卡住
- 沙箱权限过严或过松
- 长期运行成本被低估

## Labels
- 标题：生产级 Agent 系统真正依赖的不是 prompt，而是运行时能力栈
- 底部说明：系统越往上走，问题越像软件系统，而不只是模型调用

## Style Notes
- 分层关系要非常清楚
- 左侧画能力栈，右侧画故障点注释
- 强调“推理成本、状态与观测成本、运行时与部署成本”三类成本
