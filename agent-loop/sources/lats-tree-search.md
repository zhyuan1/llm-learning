# LATS: Language Agent Tree Search

来源：https://arxiv.org/abs/2310.04406
获取时间：2026-05-21

---

## Core Framework

LATS integrates Monte Carlo Tree Search with language models to create a unified approach. The framework leverages LMs' in-context learning to enable agents while incorporating "LM-powered value functions and self-reflections for proficient exploration" alongside environmental feedback loops.

## Unifying Three Capabilities

The system synergizes three distinct LM capabilities:
- **Reasoning**: Through self-reflection mechanisms
- **Acting**: Via agent decision-making processes
- **Planning**: Using Monte Carlo Tree Search for deliberate problem-solving

This addresses a key limitation where "their reliance on simple acting processes limits their broad deployment as autonomous agents."

## How the Search Tree Works

```
Root: Initial state
├── Action A (value: 0.7)
│   ├── Action A1 (value: 0.9) ← best path
│   └── Action A2 (value: 0.3)
├── Action B (value: 0.4)
│   └── ... (pruned, low value)
└── Action C (value: 0.6)
    └── ...
```

- LLM generates candidate actions (expansion)
- LLM evaluates state quality (value function)
- Self-reflection on failed paths guides future exploration
- Environmental feedback grounds the search

## Technical Approach

The search tree operates through:
1. **Selection**: UCB1 to balance exploration/exploitation
2. **Expansion**: LLM generates candidate next actions
3. **Evaluation**: LLM-based value function scores states
4. **Backpropagation**: Update values up the tree
5. **Reflection**: On failure, generate text reflection to avoid similar paths

## Performance Results

- **Programming (HumanEval)**: 92.7% pass@1 with GPT-4
- **Web navigation (WebShop)**: 75.9 average score with GPT-3.5, competitive with fine-tuned methods
- Tested on interactive QA and mathematics as well

## Cost vs Benefit

LATS is expensive (multiple LLM calls per decision step). Justified only for:
- High-value decisions where getting it wrong is costly
- Tasks with clear state evaluation criteria
- Problems where backtracking is meaningful

## Significance

LATS represents the "third generation" of agent decision-making: from simple loops (ReAct) to reflective retry (Reflexion) to deliberate search (LATS). Each generation adds computational cost but also capability ceiling.
