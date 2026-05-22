# Agent Error Recovery & Self-Correction — 多来源综合

获取时间：2026-05-21

---

## AgentDebug (UIUC/Stanford, 2025)

Introduces a "debug loop" to identify root causes (memory errors vs reflection errors) and perform targeted re-execution from failure points.

- Key mechanism: Don't restart from scratch — identify WHERE in the execution chain things went wrong, and resume from there
- Improvement: +26% success rate
- Classification: memory error (forgot something) vs reflection error (wrong conclusion from correct info)

## VIGIL: A Reflective Runtime for Self-Healing LLM Agents (2025)

arXiv:2512.07094

Monitors agent behavior, diagnoses issues as "Roses/Buds/Thorns," and generates prompt/code fixes while preserving core logic.

- Runtime monitoring (not post-hoc)
- Automatic diagnosis and fix generation
- Preserves agent's core logic while patching specific failures

## Self-Correction Risks

"The Dark Side of LLMs' Intrinsic Self-Correction":
- LLMs instructed to "double-check" may flip correct answers to wrong ones
- 58% failure rate for some models when asked to self-verify
- Implication: self-correction must be paired with external verification signals

## Error Classification Framework

| Error Type | Strategy | Example |
|-----------|----------|---------|
| Transient | Retry with exponential backoff | API timeout, rate limit |
| Semantic | Change strategy (rewrite prompt, try different approach) | Model deprecation, content filter |
| Planning | Replan from current state | Wrong decomposition |
| Memory | Re-retrieve relevant context | Context drift, forgot key fact |
| Unrecoverable | Escalate to human | Impossible task, ambiguous requirements |

## Key Design Principles

1. **Don't blindly retry** — classify errors first, adapt accordingly
2. **Targeted re-execution** — resume from failure points, don't restart entirely
3. **Beware overcorrection** — self-correction can degrade if not carefully tuned
4. **Proactive monitoring** — detect anomalies (loops, latency) early, trigger replanning or human escalation

## Brain-Inspired Recovery (Nature Communications, 2025)

Modular Agentic Planner (MAP) uses brain-inspired modules:
- Conflict monitoring (detect when current plan is failing)
- Task decomposition (break down differently after failure)
- Reduces hallucinations and infinite loops

## Seven Recovery Strategies from Real Deployments (Elevin)

1. Graceful degradation
2. Checkpoint and rollback
3. Multi-agent redundancy
4. Human escalation triggers
5. State externalization (Redis/DB for crash recovery)
6. Hierarchical context injection
7. Episodic memory for learning from past failures
