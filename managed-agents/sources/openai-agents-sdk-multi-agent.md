# OpenAI Agents SDK: Multi-Agent Orchestration

**Source:** https://openai.github.io/openai-agents-python/multi_agent/  
**Retrieved:** 2026-04-12  
**Author/Org:** OpenAI

---

## Two Primary Orchestration Approaches

### 1. LLM-Driven Orchestration
Agents use LLM intelligence for autonomous planning and decision-making. The LLM decides when to delegate, which tools to call, and how to sequence steps.

### 2. Code-Driven Orchestration
Developers control agent sequencing through explicit code logic. Makes tasks "more deterministic and predictable, in terms of speed, cost and performance."

---

## Two Primary Delegation Patterns

### Pattern A: Agents as Tools

A manager agent keeps control of the conversation and calls specialist agents through `Agent.as_tool()`.

- The new agent receives a generated input string (not full conversation history)
- The new agent is called as a tool; the conversation continues by the original agent
- Manager synthesizes the result and retains control

**Use when:** one agent should own the final answer, combine specialist outputs, or enforce shared guardrails centrally.

**Implementation:**
```python
specialist_as_tool = specialist_agent.as_tool()
manager_agent = Agent(tools=[specialist_as_tool, ...])
```

### Pattern B: Handoffs

A triage agent routes the conversation to a specialist, and that specialist becomes the active agent. The specialist receives full conversation history.

- Decentralized — the specialist takes over, not just executes and returns
- The delegated agent becomes the active participant for the rest of the turn

**Use when:** routing is part of the workflow and specialists should directly respond to users.

**Implementation:**
```python
triage_agent = Agent(handoffs=[specialist_agent_a, specialist_agent_b])
```

---

## Key Distinction

| | Agents as Tools | Handoffs |
|---|---|---|
| Who gets context | Generated task string | Full conversation history |
| Who continues | Original (manager) agent | Specialist agent |
| Control model | Centralized | Decentralized |
| Best for | Bounded subtasks, composed output | Routing workflows, direct specialist response |

---

## Code-Based Orchestration Tactics

Common code-driven patterns:
- **Structured outputs** for task classification and agent selection
- **Sequential chaining**: one agent's output becomes the next's input
- **Feedback loops**: evaluation agent reviews and iterates
- **Parallel execution**: `asyncio.gather` for independent tasks

---

## tool_use_behavior Parameter

Controls how agents process tool results:
- Default: tool results re-processed through the LLM
- `"stop_on_first_tool"`: treat first tool output as final answer
- Custom function: sophisticated result evaluation logic

---

## Cross-Agent Coordination

`RunConfig` and `RunHooks` enable monitoring and coordination across agents in a multi-agent run.
