# Building Effective AI Agents — Anthropic Engineering Blog

来源：https://www.anthropic.com/engineering/building-effective-agents
获取时间：2026-05-21

---

## Workflows vs. Agents: Core Distinction

Anthropic differentiates between two agentic system types:

**Workflows** involve "LLMs and tools orchestrated through predefined code paths," offering predictability for well-defined tasks.

**Agents** represent "systems where LLMs dynamically direct their own processes and tool usage," providing flexibility when model-driven decision-making matters.

## Workflow Patterns

**Prompt Chaining**: Decomposes tasks into sequential steps where each LLM call processes prior output. Useful for marketing copy generation followed by translation, or outline creation before document writing.

**Routing**: Classifies inputs and directs them to specialized follow-up tasks, enabling optimized prompts for distinct categories like customer service inquiries routed to different departments or models based on complexity.

**Parallelization**: Operates through two variations—sectioning (breaking tasks into independent parallel subtasks) and voting (running identical tasks multiple times). Effective for guardrails implementation or multi-perspective code reviews.

**Orchestrator-Workers**: A central LLM dynamically breaks tasks, delegates to workers, and synthesizes results. Suited for complex tasks like multi-file code changes where subtasks can't be pre-defined.

**Evaluator-Optimizer**: One LLM generates responses while another provides iterative feedback. Works best for literary translation or complex research requiring multiple refinement rounds.

## Agent Pattern

Agents operate autonomously through loops where LLMs use tools based on environmental feedback. They handle open-ended problems with unpredictable steps, requiring strong decision-making trust and extensive sandboxed testing before production deployment.

## Tool Design Recommendations

The guidance emphasizes treating agent-computer interfaces (ACI) with similar rigor as human-computer interfaces:

- Choose formats requiring minimal "thinking overhead"—formats models encounter naturally in training data
- Provide sufficient context tokens before demanding precise outputs
- Include comprehensive documentation: examples, edge cases, input requirements, and tool boundaries
- Test extensively using workbench tools to identify usage errors
- Apply "poka-yoke" principles to prevent mistakes through argument design

Anthropic noted spending more optimization time on tools than overall prompts during SWE-bench development.

## When to Use Agents vs. Workflows

Start with simple solutions. Workflows offer "predictability and consistency" for defined tasks; agents provide necessary flexibility only when complexity demonstrably improves outcomes. Agents require careful guardrails, higher costs, and potential for compounding errors—justified primarily for coding tasks, customer support, and open-ended problems requiring extensive iteration.
