# CoALA: Cognitive Architectures for Language Agents

来源：https://arxiv.org/abs/2309.02427
获取时间：2026-05-21

---

## Abstract

The paper proposes "a language agent with modular memory components, a structured action space to interact with internal memory and external environments, and a generalized decision-making process to choose actions."

## Key Framework Components

**Memory Architecture:**
The framework incorporates modular memory systems that enable agents to store and retrieve information for reasoning tasks.

- Working Memory: immediate task context (analogous to context window)
- Long-term Memory:
  - Procedural memory: how to do things (skills, procedures)
  - Semantic memory: facts and knowledge (retrieved via RAG)
  - Episodic memory: past experiences (reflections, trajectories)

**Action Space:**
Agents operate through "a structured action space to interact with internal memory and external environments," allowing both internal processing and external tool use.

- Internal actions: reasoning, retrieval, memory write
- External actions: tool calls, environment interaction, communication

**Decision-Making Process:**
A generalized decision procedure governs how agents select which actions to execute.

## Perception-Decision-Action-Feedback Mapping

The CoALA framework contextualizes language agents within cognitive science and symbolic AI traditions, enabling:

- **Perception:** Interaction with external resources (internet, APIs) and internal knowledge
- **Decision:** Generalized mechanisms choosing among available actions
- **Action:** Modulated responses affecting memory state or environment
- **Feedback:** Grounding through external resources and control flows (prompt chaining)

## Practical Impact

The paper retrospectively organizes recent language agent research and prospectively identifies development directions toward "language-based general intelligence."

## Key Insight

CoALA provides a vocabulary and structure to analyze ANY agent system. Every agent can be decomposed into: what memory it has, what actions it can take, and how it decides between them.
