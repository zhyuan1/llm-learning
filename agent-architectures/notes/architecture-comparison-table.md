# Agent Architectures Comparison Table

## 1. 主表：主流 Agent 架构横向对比

| 架构类型 | 控制权位置 | 任务分解方式 | 状态模型 | 通信方式 | 主要优势 | 主要代价 | 最适合的问题 | 不太适合的问题 | 代表框架 / 例子 |
|---|---|---|---|---|---|---|---|---|---|
| 增强型单 Agent 循环 | 主要在模型 | 基本不分解，单循环推进 | 多为进程内；可接 memory / retrieval | tool call | 最简单，起步快，调试成本最低 | 一宽一长一并行就容易吃力 | 编码助手、轻量客服、个人效率工具 | 强并行、长链路、复杂恢复任务 | OpenAI Agents SDK 基础 agent；Anthropic augmented LLM |
| 预定义工作流 / DAG / 状态机 | 主要在代码 | 静态分解，步骤预先定义 | 显式状态，可做 checkpoint | 节点间数据流 / 条件路由 | 可控、稳定、好测试，适合审计 | 灵活性较弱，需求变化时要改编排 | 审批流、文档处理、规则明确的企业流程 | 高开放度探索任务 | Anthropic workflow patterns；LangGraph 工作流 |
| Orchestrator-Worker 多 Agent | 上层 agent 定策略，下层 agent 执行 | 动态分解，由 orchestrator 按输入拆任务 | 多 agent 各自局部状态 + 汇总状态 | 任务委派 + 结果汇总 | 宽任务并行能力强，降低单 agent 上下文压力 | token 成本高，重复劳动和汇总失真风险大 | 深度研究、搜索比对、宽范围信息收集 | 强共享上下文、高耦合串行任务 | Anthropic multi-agent research system |
| Handoff / 专家接力式多 Agent | 当前 agent 判断是否转交 | 角色分解，阶段性交接 | 会话共享或部分共享 | handoff / agent-to-agent transfer | 角色边界清晰，适合前台到专家接力 | 设计不好会来回踢皮球，责任边界要清楚 | 客服分流、销售转技术、通用入口转专项处理 | 需要大量并行搜索的问题 | OpenAI Agents SDK handoffs |
| 图式 / 持久化 / 可恢复 Agent | 代码定义边界，模型在边界内决策 | 可静态也可动态，但以显式图和状态推进 | checkpoint / durable execution / persistent memory | 图节点跳转 + state update | 可恢复、可审计、适合长任务和 HITL | 工程复杂度高，建模成本更高 | 长时间运行流程、人工审批、生产级 agent | 一次性小任务、轻量原型 | LangGraph |
| 事件驱动 / 分布式多 Agent | runtime + agent 共同决定 | 多 agent 通过事件驱动协作 | 分布式状态 / runtime 管理 | message / event bus | 最强扩展性，适合跨机器跨语言协作 | 调试难、一致性复杂、系统最重 | 大型企业系统、跨服务多语言 agent 协作 | 轻量场景、快速验证阶段 | AutoGen Core / distributed runtime |
| 托管服务型 Agent Runtime | 平台 runtime 负责边界，agent 负责任务逻辑 | 编排方式可多样，重点是运行时解耦 | 外部 session / event log / sandbox state | runtime API、工具代理、执行环境隔离 | 多租户、安全边界清晰、适合产品化和托管 | 基础设施成本高，实现门槛高 | Agent 平台、SaaS、多租户托管系统 | 小团队单机场景 | Anthropic Managed Agents 思路 |

## 2. 共同点速查

| 共同点 | 说明 |
|---|---|
| 都离不开 `LLM + Tools + State` | 差别不在于有没有，而在于怎么组织 |
| 都需要明确工具接口 | 工具接口质量直接决定系统稳定性 |
| 都需要 guardrails | 包括输入输出约束、权限、停止条件、人工接管点 |
| 都需要 tracing / observability | 没有 trace，复杂 agent 系统很难维护 |
| 都在尝试解耦模型能力与系统能力 | 模型、工具、运行时都可能独立演进 |

## 3. 差异维度速查

| 差异维度 | 可选取值 | 判断问题 |
|---|---|---|
| 控制权 | 代码主导 / 模型主导 / 混合 | 下一步到底是谁决定？ |
| 分解方式 | 不分解 / 静态分解 / 动态分解 / 角色分解 | 任务是预先切好，还是现场切？ |
| 状态位置 | 进程内 / checkpoint / 外部 session / 分布式 runtime | 失败后从哪恢复？ |
| 通信方式 | tool call / handoff / shared state / message bus | agent 之间如何协作？ |
| 复杂度收益 | 灵活性 / 稳定性 / 扩展性 / 安全性 | 为什么值得更复杂？ |

## 4. 选型速查

| 如果你的任务更像… | 优先考虑 |
|---|---|
| 一个 agent 带几把工具就能完成 | 增强型单 Agent 循环 |
| 步骤固定、需要稳定和可审计 | 工作流 / DAG / 状态机 |
| 信息面很宽、适合并行探索 | Orchestrator-Worker 多 Agent |
| 角色接力比并行更重要 | Handoff |
| 任务很长，必须能恢复和插人工 | 图式 / 持久化 / 可恢复 Agent |
| 本质上是一个分布式系统问题 | 事件驱动 / 分布式多 Agent |
| 目标是平台化、托管化、多租户 | 托管服务型 Agent Runtime |

## 5. 最重要的一条经验

**先用最简单能跑通的架构。**

不要因为框架支持 multi-agent、graph、distributed runtime，就默认应该用到这些能力。只有当复杂度带来明确收益时，再升级架构。