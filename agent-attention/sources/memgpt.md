# MemGPT: Towards LLMs as Operating Systems
Authors: UC Berkeley
URL: https://arxiv.org/abs/2310.08560
Published: October 2023

## Core Idea: Virtual Context Management
Two-tier memory system:
- Main context = RAM (LLM's limited context window, active processing)
- External context = disk storage (info beyond context window, retrieved on demand)
Provides illusion of infinite context by intelligently paging between tiers.

## Memory Types
- Core memory: persistent key information
- Recall memory: conversation history
- Archival memory: external documents/data

## Key Insight
LLMs can autonomously decide what to keep in main context vs. external storage.
OS memory management principles apply: paging, interrupts, control flow.

## Why This Matters for the Article
MemGPT proves the point: the problem isn't capacity, it's SELECTION.
By making context management explicit (like an OS), agents work better.

---
# Lilian Weng - LLM Powered Autonomous Agents (2023)
URL: https://lilianweng.github.io/posts/2023-06-23-agent/

## Memory Taxonomy
- Sensory Memory → embedding representations of raw inputs
- Short-Term/Working Memory → in-context learning (bounded by context window)
- Long-Term Memory → external vector store (retrieved at query time)

## Core Limitations (verbatim from paper)
1. "The restricted context capacity limits the inclusion of historical information, detailed instructions, API call context, and responses."
2. "LLMs struggle to adjust plans when faced with unexpected errors"
3. "The reliability of model outputs is questionable"

## Key Insight
Short-term/working memory IS the context window. They are not analogous — they ARE the same thing.
This means extending context window = extending working memory capacity.
But human working memory research shows: capacity alone doesn't determine cognitive performance.
Attention, chunking, and selective encoding matter more than raw capacity.
