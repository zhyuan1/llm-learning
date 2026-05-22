# Agent Patterns Atlas — agents.kour.me

来源：http://agents.kour.me/ (redirected from kourgeorge.github.io/agents-patterns-atlas)
获取时间：2026-05-21

---

## Overview

Reusable design patterns for building long-horizon agentic AI systems — those requiring extended planning and execution sequences across multiple reasoning steps.

## Pattern Categories

### Core Workflow (4 patterns)
- **Prompt Chaining** — Sequential step coordination
- **Routing** — Directed decision-making paths
- **Parallelization** — Concurrent task execution
- **Reflection** — Evaluative feedback loops

### Tools (4 patterns)
- **Tool Use** — Agent-computer interface design
- **Shortlisting** — Tool selection optimization (relevant when many tools available)
- **Constrained Tool Use** — Restricted capability access (safety)
- **Skill Discovery** — Dynamic capability detection at runtime

### Reasoning & Planning (6 modules)
- **Task Decomposition** — Breaking complex goals into subtasks
- **Extreme Decomposition** — Granular task fragmentation (each step is atomic and verifiable)
- **Recitation** — Knowledge reinforcement
- **Prioritization** — Goal sequencing strategies
- Foundational modules on reasoning techniques and planning strategies

### Context and Memory (7 patterns)
- **Attention Engineering** — Selective focus optimization (place key info at attention-strong positions)
- **Context Editing** — Dynamic context refinement (remove stale info)
- **Filesystem as Context** — External memory utilization (files as working memory)
- **Variables Manager** — Explicit state tracking
- **RAG** — Retrieval-Augmented Generation
- Memory management and context engineering foundations

### Multi-Agent Systems (6 patterns)
- **Orchestrator-Worker** — Hierarchical coordination
- **Dynamic Agent Spawning** — Runtime agent creation
- **Planner-Checker** — Verification systems (one plans, one verifies)
- **Multi-Agent Debate** — Collaborative reasoning through disagreement
- **Swarm/Consensus Architecture** — Distributed consensus
- **Voting-Based Error Correction** — Majority validation

### Exception Handling & Human Interaction (3 patterns)
- **Exception Handling and Recovery** — Error classification and response
- **Red-Flagging** — Anomaly detection and escalation
- **Human-in-the-Loop** — Human oversight integration at decision points

### Learning and Adaptation (1+ patterns)
- **Self-Improving Agents** — Adaptive systems that learn from experience

## Key Characteristics

All patterns include runnable code examples. The atlas addresses limitations of reactive ReAct-style agents by systematizing thinking, planning, execution, and consolidation phases.

## Mapping to Four-Layer Architecture

| Layer | Relevant Patterns |
|-------|------------------|
| Perception | Attention Engineering, Context Editing, Filesystem as Context, RAG, Variables Manager |
| Decision | Task Decomposition, Extreme Decomposition, Prioritization, Routing, Prompt Chaining |
| Action | Tool Use, Shortlisting, Constrained Tool Use, Skill Discovery, Dynamic Agent Spawning |
| Feedback | Reflection, Planner-Checker, Exception Handling, Red-Flagging, Human-in-the-Loop, Self-Improving |
