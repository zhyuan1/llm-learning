---
title: "Cognitive Architectures for Language Agents (CoALA)"
authors: Sumers et al.
source: arXiv:2309.02427
date: 2023
url: https://arxiv.org/abs/2309.02427
---

## Abstract

We draw on the rich history of cognitive science and symbolic AI to propose Cognitive Architectures for Language
Agents (CoALA). CoALA describes a language agent with modular memory components, a structured action space to interact
with internal memory and external environments, and a generalized decision-making process to choose actions.

## Harness Decomposition (The CoALA Framework)

CoALA decomposes a Harness into four layers:

1. **Working Memory** — in-context information (current task, history, observations)
2. **Long-term Memory** — three subtypes:
    - Procedural: how to do things (effectively the harness code itself)
    - Semantic: factual knowledge
    - Episodic: records of past interactions
3. **Action Space** — structured into:
    - Retrieval actions (reading from memory)
    - Reasoning actions (internal computation)
    - Learning actions (updating memory)
    - External actions (interacting with environment/tools)
4. **Decision Loop** — how the agent chooses actions given current state

## Key Insight

The harness IS the procedural memory. When you write a harness, you are encoding:

- What actions the agent can take
- In what order decisions are made
- How observations are integrated into working memory

This framework gives us the vocabulary to talk about harness components precisely.
