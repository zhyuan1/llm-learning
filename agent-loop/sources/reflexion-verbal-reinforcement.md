# Reflexion: Language Agents with Verbal Reinforcement Learning

来源：https://arxiv.org/abs/2303.11366 (NeurIPS 2023)
获取时间：2026-05-21

---

## Core Mechanism

Reflexion reinforces language agents through linguistic feedback rather than weight updates. The system works by having agents "verbally reflect on task feedback signals, then maintain their own reflective text in an episodic memory buffer to induce better decision-making in subsequent trials."

## Verbal Reinforcement Learning Concept

Instead of traditional RL requiring extensive training samples and model fine-tuning, Reflexion uses natural language feedback. The framework flexibly incorporates various feedback types—scalar values or free-form language—from external or internally simulated sources.

Key distinction from traditional RL:
- No weight updates needed
- Works with frozen LLMs (API-only access)
- Feedback is stored as text in memory, not as gradient updates

## Self-Reflection Loop

The approach creates an iterative cycle:

```
Trial 1: Attempt task → Fail
         ↓
Reflect: "I failed because I didn't check X before doing Y"
         ↓
Store reflection in episodic memory
         ↓
Trial 2: Read past reflections → Adjust approach → Attempt task
         ↓
Success (or reflect again, up to N trials)
```

## Performance Results

- **HumanEval (Coding)**: 91% pass@1 accuracy (previous SOTA GPT-4: 80%)
- **AlfWorld (Decision-making)**: Significant improvement in sequential task completion
- **HotpotQA (Reasoning)**: Improved multi-hop question answering

## Risks and Limitations

- Over-reflection: models told to "double-check" can flip correct answers to wrong (58% degradation rate in some studies)
- Engineering practice: limit reflection rounds to 2-3, distinguish "low confidence" from "confirmed wrong"
- External verification (tests, compilers) is more reliable than pure introspection

## Significance

Reflexion proves that agents can learn from experience WITHOUT fine-tuning — using language as the medium of learning. This is the foundation of the feedback layer in agent architecture.
