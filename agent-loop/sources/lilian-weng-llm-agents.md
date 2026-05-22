# LLM Powered Autonomous Agents — Lilian Weng

来源：https://lilianweng.github.io/posts/2023-06-23-agent/
获取时间：2026-05-21

---

## Agent System Overview

Agent = LLM + Planning + Memory + Tool Use

## Memory Architecture

**Short-term Memory**: Treated as "in-context learning" constrained by transformer context windows. Handles immediate task information needed for current operations.

**Long-term Memory**: External vector stores enable agents to "retain and recall (infinite) information over extended periods." Uses embedding representations stored in searchable databases.

**Sensory Memory**: The initial stage processing raw inputs across modalities before encoding into usable representations.

## Memory Retrieval (MIPS)

Fast retrieval uses approximate nearest neighbors algorithms:
- LSH (Locality-Sensitive Hashing)
- ANNOY (Approximate Nearest Neighbors Oh Yeah)
- HNSW (Hierarchical Navigable Small World)
- FAISS (Facebook AI Similarity Search)
- ScaNN (Scalable Nearest Neighbors)

## Tool Use Design Patterns

- **Modular routing**: LLMs act as routers directing queries to specialized expert modules (MRKL systems)
- **API augmentation**: Fine-tuning models to learn when and how to invoke external tools
- **Task planning**: Using LLMs to decompose complex requests into sequential API calls with proper dependencies
- **Multi-stage workflows**: Planning → model selection → execution → response generation

## Planning Methods

**Task Decomposition**:
- Chain of Thought (CoT): step-by-step reasoning
- Tree of Thoughts (ToT): multi-path exploration with evaluation
- External classical planners (PDDL, etc.)

**Self-Reflection**:
- ReAct: interleave reasoning and acting
- Reflexion: verbal self-critique stored in memory
- Chain of Hindsight: learn from sequence of past outputs

## Key Challenges Identified

1. Finite context length — hard to include full history
2. Long-term planning difficulty — error accumulation
3. Reliability of natural language interfaces — formatting errors, tool misuse
4. Hallucination in reasoning chains

## Significance

This blog post is the canonical survey that organized the field in 2023. It established the Planning + Memory + Tool Use decomposition that most subsequent work builds on.
