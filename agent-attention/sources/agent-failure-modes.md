# Agent Failure Modes in Long-Horizon Tasks

## Goal Drift (3 root causes identified)
1. Tool-return semantic dilution: intermediate results lose connection to original goal
2. Subtask anchoring: agents fixate on early strategies even as conditions change  
3. Recency bias: recent tokens dominate attention, agent "forgets" initial instructions

## Context Pollution
- Errors/hallucinations from early steps persist and compound through long context
- Corrupts later reasoning
- Example: SWE-bench agents failing multi-file coordination tasks

## Context Rot
- Performance degrades sharply when context utilization exceeds ~40% of window capacity
- Adding more context adds more noise, which dilutes signal-to-noise ratio

## Safety Instability (arXiv:2512.02445)
- Refusal rates fluctuate unpredictably in long contexts
- GPT-4.1-nano refusal rate jumps erratically 5% → 40% at 200K tokens
- Safety alignment does NOT scale linearly with context size

## The Fundamental Problem
LLM attention is probabilistic, not deterministic.
Agent tasks require reliable state across 100+ tool calls.
This is a RELIABILITY problem, not a CAPACITY problem.

---
# Anthropic Engineering: Effective Context Engineering for AI Agents
URL: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

## Core Framing (Anthropic's own words)
"What configuration of context is most likely to generate our model's desired behavior?"

## Key Admission
"Context rot" — performance degrades as context grows, caused by transformer's n² token relationships.
"Models remain highly capable at longer contexts but may show reduced precision"
= Anthropic explicitly acknowledges the problem even in their own models.

## Context Engineering vs Prompt Engineering
Prompt engineering: crafting effective instructions (single-turn)
Context engineering: managing all information across multiple turns — tools, history, external data

## Practical Mitigations (from practitioners building agents)
1. Compaction — summarize conversations nearing context limits; preserve decisions, discard redundant outputs
2. Structured Note-Taking — agents maintain NOTES.md for persistent tracking across context resets
3. Sub-Agent Architectures — specialized agents with focused tasks, return condensed summaries to main agent
4. Just-in-time retrieval — maintain lightweight identifiers, dynamically load data when needed

## Key Quote on Tools
"If a human engineer can't definitively say which tool should be used, agents will struggle similarly."
= Tool-use degradation correlates with context-overloaded state

## Significance for Article
This is Anthropic's own engineering blog — not academic paper, real-world practitioner confession.
They are not solving context rot by making the window bigger.
They are solving it by managing what goes INTO the window.

---
# Karpathy's OS Analogy — Software 3.0
Source: Multiple talks 2025, including Sequoia Ascent 2026

## The Analogy
LLM Core Model = CPU
Context Window = RAM  
API/Tools = System Calls

## Key Implication for Our Article
RAM (context window) is volatile and finite.
The OS (agent architecture) decides what to load into RAM, when, and for how long.
A bigger context window = bigger RAM.
But nobody argues bigger RAM alone makes your OS smarter.
The intelligence is in the memory management, not the capacity.

## Context Engineering Challenges (Karpathy's terms)
- Context Distraction: overloading dilutes model focus
- Context Poisoning: errors propagate across interactions via retained context
