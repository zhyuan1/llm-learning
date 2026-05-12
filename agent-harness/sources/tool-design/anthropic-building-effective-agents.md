---
title: "Building Effective Agents"
authors: Anthropic Engineering
source: anthropic.com/engineering/building-effective-agents
date: 2024
url: https://www.anthropic.com/engineering/building-effective-agents
---

## Core Architecture

Two foundational patterns:

- **Workflows**: LLMs + tools orchestrated through predefined code paths
- **Agents**: LLM dynamically controls tool selection and execution sequences

## Tool Design Principles (Critical)

> "Tool specifications deserve just as much prompt engineering attention as your overall prompts."

Key rules:

1. **Format optimization**: Choose formats natural to LLM training data. Avoid unnecessary escaping, counting overhead
2. **Documentation quality**: Include usage examples, edge cases, input requirements, clear boundaries between similar
   tools — "write a great docstring for a junior developer"
3. **Poka-yoke design**: Structure arguments to prevent misuse
    - Example: requiring **absolute filepaths** eliminated model errors with relative paths after directory changes (
      SWE-bench finding)
4. **Testing**: Run many example inputs to see what mistakes the model makes, iterate

## Execution Loop Design

Agents operate in feedback loops:

1. Receive task
2. Plan independently
3. Execute → get ground truth from environment (tool results, code execution)
4. Pause at checkpoints or blockers
5. Include stopping conditions

## Practical Failure Modes

- **Over-complexity**: Adding agentic patterns without measurable performance improvement
- **Abstraction hiding**: Framework layers obscure prompts/responses, hindering debugging
- **Compounding errors**: Autonomous agents → higher costs → compounding errors

## Multi-Agent Orchestration

- Orchestrator-workers: central LLM decomposes tasks, workers execute
- Parallelization: sectioning (independent parallel subtasks) or voting (multiple attempts)
- Trust: subagents should be treated as untrusted inputs — validate their outputs
