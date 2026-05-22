# ReAct: Reasoning and Acting in Language Models

来源：https://arxiv.org/abs/2210.03629 (ICLR 2023)
获取时间：2026-05-21

---

## Core Mechanism

The ReAct approach generates both reasoning traces and task-specific actions in an interleaved pattern:

```
Thought: I need to find information about X
Action: Search[X]
Observation: [search results]
Thought: Based on the results, I should...
Action: Lookup[specific detail]
Observation: [detail found]
Thought: Now I have enough to answer
Action: Finish[answer]
```

## Why Interleaving Helps

"Reasoning traces help the model induce, track, and update action plans as well as handle exceptions, while actions allow it to interface with external sources, such as knowledge bases or environments, to gather additional information."

This combination overcomes "issues of hallucination and error propagation prevalent in chain-of-thought reasoning."

Key insight: reasoning without acting = hallucination risk; acting without reasoning = blind execution. ReAct combines both.

## Performance Results

**HotpotQA and Fever:**
- Overcomes hallucination by grounding reasoning in real retrieved information
- More interpretable than baseline methods
- Hallucination rate: 56% → ~0%

**ALFWorld and WebShop:**
- Outperforms imitation and reinforcement learning by absolute success rate of 34% and 10% respectively
- Uses only one or two in-context examples (few-shot)

## Significance for Agent Architecture

ReAct established the foundational execution loop that nearly all modern agents use. The Thought-Action-Observation cycle is the "dumb loop" that Anthropic references — the loop itself is simple, all intelligence is in the model's reasoning within the loop.
