# SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering

来源：https://arxiv.org/abs/2405.15793 (Princeton, 2024)
获取时间：2026-05-21

---

## Agent-Computer Interface (ACI) Concept

The paper introduces ACI as a specialized interface designed for language model agents rather than humans. The authors argue that "LM agents represent a new category of end users with their own needs and abilities, and would benefit from specially-built interfaces."

Key insight: just as HCI (Human-Computer Interaction) designs interfaces for humans, ACI designs interfaces for LLMs. The two have different needs:
- Humans: visual, spatial, interactive
- LLMs: textual, sequential, context-window-constrained

## Design Choices That Mattered

The custom ACI "significantly enhances an agent's ability to create and edit code files, navigate entire repositories, and execute tests and other programs."

Specific design decisions:
- **File viewer**: shows code with line numbers in a paginated window (not entire files)
- **Edit command**: requires specifying exact line ranges (prevents accidental overwrites)
- **Search**: returns structured results with file paths and line numbers
- **Error output**: formatted to highlight the actual error vs boilerplate

## Critical Insight

"Interface design affects the performance of language model agents" — this is the central finding. The same model (GPT-4) with better interface design outperforms the same model with standard tools by 3x+.

This means: optimizing how agents interact with systems matters MORE than traditional prompt engineering.

## Performance Results

- SWE-bench: 12.5% pass@1 (previous non-interactive LMs: 3.8%)
- HumanEvalFix: 87.7%
- Same model, same training — only the interface changed

Later results with Claude 3.5 Sonnet + evolved harness: 49% on SWE-bench Verified.

## Implications for Perception Layer

SWE-agent proves that the perception layer (what the model sees) is the single highest-leverage design decision in an agent system. Before optimizing decision-making or adding more tools, optimize the observation format.
