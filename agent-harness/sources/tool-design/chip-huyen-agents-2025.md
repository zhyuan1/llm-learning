---
title: "Agents — Chip Huyen"
authors: Chip Huyen
source: huyenchip.com
date: 2025-01-07
url: https://huyenchip.com/2025/01/07/agents.html
---

## Core Agent Components

Three elements: **environment** + **action set** (tools) + **AI brain** (planning/execution)

## Tool Categories

1. **Knowledge augmentation**: retrievers, SQL executors, APIs
2. **Capability extension**: calculators, code interpreters, timezone converters, multimodal bridges
3. **Write actions**: database modifiers, email APIs, banking transfers (highest risk, needs safeguards)

Tool use can boost model performance by **11-17%** on benchmarks vs. just prompting.

## Execution Loop Architecture

Decoupled pattern:
> Plan → Validate → Execute → Reflect

Control flows: sequential, parallel, conditional (if/else), iterative (loops)

## Critical Performance Data

- **Compound accuracy problem**: 95% per-step accuracy → only **60% over 10 steps**
  (formula: 0.95^10 = 0.60)
- Planning granularity tension: higher-level plans easier to generate; lower-level plans easier to execute
- Tool inventory: more tools increase capability but reduce selection efficiency

## Observation/Context Accumulation

Task history accumulates tool outputs at each step. Reflection mechanisms (self-critique or separate evaluators) analyze
outcomes before proceeding.

## Model-Tool Preference Differences

- GPT-4: favors knowledge retrieval tools
- ChatGPT: favors image captioning tools

This suggests observation format and tool descriptions must be tuned per model.
