# OpenAI Agents SDK — Multi-Agent Handoff Orchestration (2025)

来源：OpenAI documentation / web search synthesis
获取时间：2026-05-21

---

## Core Concepts

The OpenAI Agents SDK enables dynamic task delegation between specialized agents through **handoffs** — a decentralized mechanism where agents autonomously transfer control based on context.

Key features:
- **Decentralized Workflows**: Agents directly hand off tasks to peers (unlike centralized supervisor patterns)
- **Context Preservation**: Full conversation history and tool outputs are passed during handoffs
- **Agents as Tools**: Specialized agents can be exposed as tools (e.g., `agent.as_tool()`)

## Orchestration Strategies

| Strategy | Description |
|----------|-------------|
| **Deterministic** | Explicit rules route tasks (e.g., keyword-based routing) |
| **Dynamic (LLM-Driven)** | A "triage agent" uses LLM reasoning to delegate adaptively |
| **Sequential** | Default turn-based execution |
| **Parallel** | Concurrent execution via `asyncio.gather()` |

## Code Example

```python
from agents import Agent, Runner

# Define specialized agents
researcher = Agent(name="Researcher", instructions="Search and analyze data.")
translator = Agent(name="Translator", instructions="Translate text.")

# Triage agent with handoff options
triage_agent = Agent(
    name="Triage",
    instructions="Route queries dynamically.",
    handoffs=[researcher, translator]
)

# Run: Handoff occurs based on LLM reasoning
response = await Runner.run(triage_agent, input="Research climate change, then translate to French.")
```

## Key Design Decisions

- **Handoffs vs Tools**: Handoff = full control transfer; Tool = bounded subtask, returns result to caller
- **Context preservation**: entire conversation history travels with the handoff
- **Guardrails**: input guardrails (on first agent), output guardrails (on final output), tool guardrails (per tool call)
- **Tripwire mechanism**: trigger = immediate agent stop

## Evolved from Swarm

The SDK evolved from OpenAI's experimental Swarm framework, now with:
- Enterprise-grade async support
- MCP tool integration
- Built-in tracing for observability

## Relevance to Four-Layer Architecture

- **Perception**: Context preservation during handoffs ensures continuity
- **Decision**: Triage agent implements routing (a decision-layer pattern)
- **Action**: Agents-as-tools pattern = bounded action delegation
- **Feedback**: Guardrails and tripwires = feedback mechanisms that can halt execution
