# Anthropic: Building Effective Agents

**Source:** https://www.anthropic.com/engineering/building-effective-agents  
**Retrieved:** 2026-04-12  
**Author/Org:** Anthropic Engineering

---

## Core Distinction: Workflows vs. Agents

- **Workflows:** LLMs and tools orchestrated through predefined code paths
- **Agents:** Systems where LLMs dynamically direct their own processes and tool usage

---

## Key Orchestration Patterns

### 1. Prompt Chaining
Decomposes tasks into sequential steps where each LLM call processes the previous output. Developers can insert programmatic validation checks at intermediate stages.

**Best for:** Tasks that decompose cleanly into fixed subtasks; when latency tradeoff is acceptable for improved accuracy.

### 2. Routing
Classifies inputs and directs them to specialized downstream handlers. Enables "separation of concerns, and building more specialized prompts" rather than optimizing a single response path.

### 3. Parallelization
Two variations:
- **Sectioning:** Breaking independent subtasks for concurrent execution
- **Voting:** Running identical tasks multiple times for diverse outputs (increases confidence)

**Best for:** Subtasks that can execute simultaneously, or when multiple perspectives improve output quality.

### 4. Orchestrator-Workers Pattern
"A central LLM dynamically breaks down tasks, delegates them to worker LLMs, and synthesizes their results."

**Key difference from parallelization:** Subtasks are determined dynamically based on specific inputs, not predefined — essential for unpredictable problem spaces like coding.

### 5. Evaluator-Optimizer
One LLM generates responses while another provides iterative feedback.

**Best for:** Scenarios with clear evaluation criteria where feedback demonstrably improves outputs.

---

## When to Use Multi-Agent Approaches

**Core principle: start simple. Only increase complexity when needed.**

Multi-agent systems "often trade latency and cost for better task performance."

Implement complexity only when simpler solutions demonstrably fail. Prioritize measurement and iteration throughout.

---

## Key Engineering Insight

The orchestrator-workers pattern is not the same as parallelization. In parallelization, the structure is predetermined. In orchestrator-workers, the central LLM decides the breakdown dynamically. This matters in practice: you can't enumerate every possible subtask decomposition upfront for open-ended tasks.

---

## Practical Rules of Thumb

1. Prefer single-agent with good tools over multi-agent when possible
2. Add multi-agent complexity only when single agent demonstrably fails
3. Measure the performance delta before committing to architectural complexity
4. In multi-agent systems, each agent's context should only contain what that agent needs
