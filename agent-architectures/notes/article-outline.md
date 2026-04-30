# Article Outline

## 暂定标题

1. **Agent 架构不是一棵树：从单循环到多智能体，再到运行时系统**
2. **谁在决定下一步？主流 Agent 架构的分歧、共识与适用边界**
3. **别再只问“用哪个框架”：Agent 主流架构的真正分类方法**

## 一句话中心论点

主流 Agent 架构的分歧，表面上看是单 Agent 和多 Agent 的区别，实质上更接近两条主轴的不同折中：

1. **控制权在代码还是模型**
2. **系统重心在 prompt 编排还是 runtime 能力**

## 目标读者

- 正在选 Agent 技术路线的工程师
- 想把 Agent 从 demo 做到生产的团队
- 容易把“框架差异”误当成“架构差异”的读者

## 文章结构

### 0. 开篇：为什么“单 Agent vs 多 Agent”这个问法不够好
**本节目标**
- 先打破最常见的二分法
- 把读者从“框架名词表”拉回到“架构判断问题”上

**要点**
- 很多讨论把框架、编排模式、运行时能力混在一起
- 真正的关键不是 agent 数量，而是谁控制流程、状态如何存在、系统如何恢复和扩展
- 提出全文的两条主轴：控制权、运行时重心

**来源**
- `deep-research-digest.md`
- `quick-reference.md`
- `architecture-comparison-table.md`

---

### 1. 第一层分类：控制权到底在代码还是模型
**本节目标**
- 建立全文最基础的分类轴
- 给后文所有架构形态一个统一坐标系

**要点**
- Code-first：workflow / DAG / graph / flows
- Model-first：autonomous agent / 单 agent loop
- Hybrid：代码定义边界，模型在边界内动态决策
- 为什么 Hybrid 会成为很多生产系统的现实折中

**来源**
- `anthropic-building-effective-agents-notes.md`
- `langgraph-notes.md`
- `openai-agents-sdk-notes.md`
- `deep-research-digest.md`

---

### 2. 最基础的形态：增强型单 Agent 循环
**本节目标**
- 说明为什么它仍然是大多数系统的起点
- 强调“简单但够用”不是退而求其次，而是工程选择

**要点**
- 基本结构：LLM + tools + retrieval/memory
- 优势：简单、低成本、易调试
- 失效边界：任务太宽、太长、太并行、需要恢复
- 为什么很多系统一开始不该直接上 multi-agent

**来源**
- `anthropic-building-effective-agents-notes.md`
- `openai-agents-sdk-notes.md`
- `quick-reference.md`

---

### 3. 从“循环”到“流程”：Workflow、Graph、Flows
**本节目标**
- 解释为什么很多系统真正需要的不是更多 agent，而是更强编排
- 把 workflow / graph / flows 放到同一谱系里讲清楚

**要点**
- Prompt chaining、routing、parallelization、evaluator-optimizer
- LangGraph：显式 state、durable execution、HITL
- CrewAI：Flows 负责 orchestration，Crews 负责执行
- 共同点：路径更显式、状态更清楚、可恢复性更强

**来源**
- `anthropic-building-effective-agents-notes.md`
- `langgraph-readme.md`
- `langgraph-notes.md`
- `crewai-flows-notes.md`
- `crewai-readme.md`

---

### 4. 多 Agent 的第一条主线：Manager、Handoff 和 Orchestrator-Worker
**本节目标**
- 把常见的多 Agent 协作方式拆开，不再笼统叫 multi-agent
- 说明这些模式适合的不是同一种问题

**要点**
- Agents as tools：manager 始终掌控
- Handoff：specialist 直接接管
- Orchestrator-worker：上层拆任务，下层并行执行
- 三者在控制权、用户面、一致性、并行能力上的差别

**来源**
- `openai-agent-orchestration-notes.md`
- `openai-agents-sdk-notes.md`
- `anthropic-multi-agent-research-system-notes.md`
- `deep-research-digest.md`

---

### 5. 多 Agent 的第二条主线：Conversation-based 到 Event-driven Runtime
**本节目标**
- 说明为什么 AutoGen 这一路和 workflow/graph 不是一回事
- 展示多 agent 架构如何从“对话式协作”演进到“运行时系统”

**要点**
- AutoGen 论文：conversation as coordination mechanism
- AutoGen runtime：event-driven、多 agent、distributed
- Microsoft Agent Framework：graph workflows、checkpoint、observability、hosting
- 这条路线的本质：从实验式协作走向企业级 orchestration/runtime

**来源**
- `autogen-paper-notes.md`
- `microsoft-autogen-notes.md`
- `microsoft-autogen-readme.md`
- `microsoft-agent-framework-notes.md`
- `microsoft-agent-framework-readme.md`

---

### 6. 真正的分水岭：运行时能力开始成为主角
**本节目标**
- 把讨论从 prompt 编排推进到 runtime 设计
- 解释为什么生产系统越来越像“应用平台”而不是“提示词拼装”

**要点**
- state / session 外置
- checkpoint 与 failure recovery
- sandbox / credential isolation
- human-in-the-loop
- tracing / observability / middleware / deployment
- Anthropic Managed Agents 和 Microsoft Agent Framework 所体现的方向

**来源**
- `deep-research-digest.md`
- `microsoft-agent-framework-notes.md`
- `anthropic-multi-agent-research-system-notes.md`
- `langgraph-notes.md`

---

### 7. 适用范围：每种架构到底什么时候开始失效
**本节目标**
- 不只讲优点，还要讲边界
- 帮读者建立“什么时候不该用”的判断力

**要点**
- 单 agent loop 的失效边界
- workflow / graph / flows 的失效边界
- handoff 的失效边界
- orchestrator-worker 的失效边界
- conversation-based multi-agent 的失效边界
- distributed runtime 的失效边界

**来源**
- `quick-reference.md`
- `architecture-comparison-table.md`
- `deep-research-digest.md`

---

### 8. 框架地图：不要再把框架选择当成架构选择
**本节目标**
- 把前面的抽象分析落到具体框架
- 帮读者避免“看见框架就当路线”的误区

**要点**
- Anthropic：taxonomy + orchestrator-worker 实践
- OpenAI Agents SDK：primitives + manager/handoff
- LangGraph：stateful graph + recovery + HITL
- AutoGen：conversation-based → runtime
- Microsoft Agent Framework：企业级 graph/runtime
- CrewAI：workflow orchestration + crews execution

**来源**
- `SOURCES.md`
- `architecture-comparison-table.md`
- `deep-research-digest.md`

---

### 9. 结尾：Agent 架构不是一棵树，而是一个矩阵
**本节目标**
- 收束全文
- 把分类方法重新压缩成一个可复用判断框架

**要点**
- 主流架构不是纯粹形态，而是控制权与 runtime 重心的不同折中
- 真正的工程问题不是“哪个框架最强”，而是“当前任务需要哪种控制方式和运行时能力”
- 给出一句最终建议：先用最简单能跑通的架构，再按明确收益升级

**来源**
- `deep-research-digest.md`
- `quick-reference.md`

## 附：建议配套图表

1. **二维架构地图**
   - 横轴：控制权（代码 → 模型）
   - 纵轴：系统重心（prompt 编排 → runtime 能力）
   - 把 OpenAI Agents SDK、LangGraph、AutoGen、CrewAI、Microsoft Agent Framework、Managed Agents 放进去

2. **架构对比表**
   - 可直接复用 `architecture-comparison-table.md`

3. **演进路线图**
   - 单 agent → workflow/graph
   - 单 agent → handoff/manager
   - AutoGen → Microsoft Agent Framework
   - workflow orchestration + agent execution 双层化（CrewAI）

## 下一步建议

如果继续 Phase 4，可以按这份提纲先写：
1. 开篇
2. 第一层分类
3. 单 agent / workflow / multi-agent 三大主体章节
4. 运行时章节
5. 适用边界与结尾