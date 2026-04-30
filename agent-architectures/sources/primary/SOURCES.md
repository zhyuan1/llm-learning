# Primary Sources Index

## 研究日期

2026-04-30

## 使用原则

优先记录官方博客、官方文档、官方仓库与原始论文；避免把二手解读当作架构事实来源。

## 清单

### 1. Anthropic — Building Effective AI Agents
- URL: https://www.anthropic.com/research/building-effective-agents
- 类型: 官方研究/实践文章
- 本地记录: `anthropic-building-effective-agents-notes.md`
- 价值: 提供了最清晰的 workflow vs. agent 区分，以及 prompt chaining / routing / parallelization / orchestrator-workers / evaluator-optimizer 等基本 taxonomy

### 2. Anthropic Engineering — How we built our multi-agent research system
- URL: https://www.anthropic.com/engineering/multi-agent-research-system
- 类型: 官方工程实践文章
- 本地记录: `anthropic-multi-agent-research-system-notes.md`
- 价值: 给出了 production multi-agent system 的 orchestrator-worker 分工方式、并行策略、成本和失败模式

### 3. OpenAI Agents SDK
- URL: https://openai.github.io/openai-agents-python/
- 类型: 官方文档 + 官方仓库
- 本地记录:
  - `openai-agents-sdk-notes.md`
  - `openai-agents-sdk-readme.md`
- 价值: 展示 agent / tools / handoffs / guardrails / traces 这一组最小 primitives

### 4. OpenAI Agents SDK — Agent Orchestration
- URL: https://openai.github.io/openai-agents-python/multi_agent/
- 类型: 官方文档
- 本地记录: `openai-agent-orchestration-notes.md`
- 价值: 明确区分 agents as tools、handoffs、LLM-based orchestration、code-based orchestration 等多 agent 控制模型

### 5. LangGraph
- URL: https://github.com/langchain-ai/langgraph
- 类型: 官方仓库 README
- 本地记录:
  - `langgraph-readme.md`（README 原始副本）
  - `langgraph-notes.md`（提取笔记）
- 价值: 代表 graph/state-machine + persistence + human-in-the-loop + durable execution 这一派

### 6. Microsoft AutoGen
- URL: https://microsoft.github.io/autogen/stable/index.html
- 类型: 官方文档 + 官方仓库 + 原始论文
- 本地记录:
  - `microsoft-autogen-notes.md`
  - `microsoft-autogen-readme.md`
  - `autogen-paper-notes.md`
- 价值: 代表 event-driven、conversation-based、distributed multi-agent runtime 的方向

### 7. Microsoft Agent Framework
- URL: https://github.com/microsoft/agent-framework
- 类型: 官方仓库 README / 官方框架说明
- 本地记录:
  - `microsoft-agent-framework-readme.md`
  - `microsoft-agent-framework-notes.md`
- 价值: 展示 AutoGen 向企业级 graph workflow + observability + deployment runtime 演进的路线

### 8. CrewAI — Flows
- URL: https://docs.crewai.com/en/concepts/flows
- 类型: 官方文档 + 官方仓库
- 本地记录:
  - `crewai-flows-notes.md`
  - `crewai-readme.md`
- 价值: 代表 workflow orchestration + crews execution 的双层结构

## 备注

- 已保存的原始副本包括：LangGraph README、OpenAI Agents SDK README、Microsoft AutoGen README、Microsoft Agent Framework README、CrewAI README
- Anthropic 官方网页来源在当前本地 bash 抓取路径上未稳定获取完整正文副本，因此保留了**来源 URL + 提取笔记**；后续如果需要做长期归档，可补充浏览器导出或更稳定的网页镜像
- 当前资料池已经覆盖了单 agent loop、workflow/graph、orchestrator-worker、handoff、多 agent conversation、event-driven distributed runtime、managed/runtime-oriented 这几条主线