---
title: "LLM Powered Autonomous Agents"
authors: Lilian Weng (OpenAI)
source: lilianweng.github.io
date: 2023-06-23
url: https://lilianweng.github.io/posts/2023-06-23-agent/
---

## Three Essential Components

1. **Planning**: decomposition into subgoals (Chain-of-Thought, Tree-of-Thoughts, ReAct)
2. **Memory**:
    - Short-term: in-context learning within finite context window
    - Long-term: external vector stores with fast retrieval
3. **Tool Use**: calling external APIs for information missing from model weights

## Critical Breaking Points (Failure Modes)

1. **Finite context limitation**: Restricted capacity constrains historical info, detailed instructions, and the ability
   to learn from mistakes through self-reflection
2. **Long-term planning difficulties**: "LLMs struggle to adjust plans when faced with unexpected errors, making them
   less robust compared to humans"
3. **Natural language interface reliability**: The system depends on parsing model outputs → formatting errors, "
   rebellious behavior"

## Key Insight for Harness Design

The three breaking points directly map to three harness responsibilities:

- Context management → working memory design
- Error recovery → loop + feedback design
- Output parsing → observation format design

Harness is not a passive container; it is the active mitigation system for all three failure modes.
