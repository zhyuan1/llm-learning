# Research Log

## 研究主题

分析 Agent 存在的主流架构、差异与相同，以及每种架构的适用范围。

## 研究模式

### 阶段 1：Quick Reference
- 目标：快速建立可操作的心智模型，不以成文文章为目标
- 日期：2026-04-30

### 阶段 2：Deep Research
- 目标：扩展一手资料池，增强对不同架构路线的覆盖，并把来源和过程完整落盘
- 日期：2026-04-30

## 工作步骤

### 1. 预检查

检查了 `/read` 与 `/write` skill：均已安装。

### 2. Quick Reference 模式确认

最初在 Learn 的三种模式里选择了 **Quick Reference**，因为当时目标是先建立心智模型，而不是直接写长文。

### 3. Quick Reference 阶段的一手资料发现

优先选择官方博客、官方文档与官方仓库，避免二手解读。第一轮采用的资料包括：

1. Anthropic 对 workflow vs. agent 的官方 taxonomy
2. Anthropic 的生产级 multi-agent research system 经验
3. OpenAI Agents SDK 的官方 primitives
4. LangGraph 的官方仓库/README
5. Microsoft AutoGen 的官方文档

### 4. Quick Reference 阶段的提取维度

每份资料统一按以下问题提取：

- 核心抽象是什么
- 控制权在代码还是模型
- 多 agent 如何分工或通信
- 状态如何持久化
- 是否支持恢复、人类介入、分布式扩展
- 适用于什么问题，不适用于什么问题

### 5. Quick Reference 阶段的归纳方式

最终把 Agent 架构分成两层来理解：

#### A. 推理 / 编排层
- 单 Agent 循环
- 预定义工作流 / DAG / 状态机
- Orchestrator-Worker 多 Agent
- Handoff / 专家接力式多 Agent
- 事件驱动 / 分布式多 Agent

#### B. 运行时 / 基础设施层
- 进程内状态
- 持久化 checkpoint
- 外部 session / event log
- 执行环境隔离（sandbox）
- tracing / guardrails / HITL

### 6. 切换到 Deep Research

随后继续进入 **Deep Research**，目标从“快速建立模型”升级为“扩展资料池并增强来源覆盖”。

Deep Research 阶段重点补充了：

1. OpenAI Agents SDK 的多 agent orchestration 文档
2. AutoGen 原始论文
3. Microsoft Agent Framework 官方仓库与架构说明
4. CrewAI Flows 官方文档与官方仓库 README
5. 多个官方仓库 README 原始副本，用作本地一手资料镜像

### 7. Deep Research 阶段的新增判断

新增资料帮助补强了几个关键认识：

1. **多 agent 不等于一种架构**：至少要区分 manager-as-tool、handoff、conversation-based、event-driven 这些不同协作模型
2. **workflow 与 multi-agent 经常是叠加关系**：不少系统其实是“上层 workflow，下层 agent 执行”
3. **企业化方向会把架构重点从 prompt 编排转向 runtime 能力**：包括 checkpoint、observability、deployment、middleware、sandbox、HITL
4. **框架演进本身也是架构演进线索**：例如 AutoGen → Microsoft Agent Framework，体现了从实验型多 agent 走向企业级 orchestration/runtime 的趋势

## 主要判断原则

1. 先选最简单能跑通的架构
2. 只有在收益明确时才引入更复杂的多 Agent 或分布式设计
3. 先判断任务类型，再判断框架，不要反过来
4. 多 Agent 适合“可并行探索”的宽任务，不适合强共享上下文任务
5. 企业生产系统通常会逐步从单 agent loop 走向 graph/stateful runtime，而不是一步上最复杂形态

## 产出物

- `quick-reference.md`：结构化笔记
- `architecture-comparison-table.md`：架构横向对比表
- `article-outline.md`：带来源映射的文章提纲
- `article-draft.md`：按提纲写出的正文初稿
- `deep-research-digest.md`：深度研究阶段的 taxonomy 与演进线索摘要
- `../sources/primary/SOURCES.md`：一手资料索引
- `../sources/primary/*.md`：一手资料本地记录与部分原始副本

## 备注

- 已保存多份官方仓库 README 原始副本，便于长期归档
- Anthropic 官方网页来源在本地网络环境下未稳定拿到可保存的完整正文副本，因此保存了**来源 URL + 提取笔记**
- 当前资料池已经足够支撑下一步做更系统的 digest / outline / survey 写作