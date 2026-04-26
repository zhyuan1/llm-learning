# smolagents: Multi-Agents (Guided Tour)

**Source:** https://huggingface.co/docs/smolagents/main/en/guided_tour#multiagents  
**Retrieved:** 2026-04-12  
**Author/Org:** HuggingFace (smolagents team)

---

## Multi-agents

Multi-agent systems have been introduced with Microsoft's framework Autogen (https://huggingface.co/papers/2308.08155).

In this type of framework, you have several agents working together to solve your task instead of only one. It empirically yields better performance on most benchmarks. The reason for this better performance is conceptually simple: for many tasks, rather than using a do-it-all system, you would prefer to specialize units on sub-tasks. Here, having agents with separate tool sets and memories allows efficient specialization. For instance, why fill the memory of the code generating agent with all the content of webpages visited by the web search agent? It's better to keep them separate.

You can build hierarchical multi-agent systems with `smolagents`.

To do so, ensure your agent has `name` and `description` attributes, which will then be embedded in the manager agent's system prompt to let it know how to call this managed agent, as we also do for tools. Then pass this managed agent in the parameter `managed_agents` upon initialization of the manager agent.

### Minimal example

```python
from smolagents import CodeAgent, InferenceClientModel, WebSearchTool

model = InferenceClientModel()

web_agent = CodeAgent(
    tools=[WebSearchTool()],
    model=model,
    name="web_search_agent",
    description="Runs web searches for you. Give it your query as an argument."
)

manager_agent = CodeAgent(
    tools=[], model=model, managed_agents=[web_agent]
)

manager_agent.run("Who is the CEO of Hugging Face?")
```

---

## Agent Types: CodeAgent vs ToolCallingAgent

### CodeAgent
- Generates tool calls as Python code snippets
- Code is executed locally or in a secure sandbox
- Tools are exposed as Python functions
- Strengths: highly expressive, flexible, emergent reasoning for multi-step problems
- Limitations: risk of syntax errors, less predictable, requires secure execution environment

### ToolCallingAgent
- Writes tool calls as structured JSON
- Common format used in many frameworks (OpenAI API)
- Strengths: reliable, safe (arguments strictly validated), interoperable
- Limitations: low expressivity, must predefine all actions, no code synthesis

### When to use which
- CodeAgent: reasoning, chaining, dynamic composition, problem-solver
- ToolCallingAgent: simple atomic tools, high reliability, dispatcher/controller

---

## How the managed_agents system prompt injection works

When a managed agent is registered, its `name` and `description` are injected into the manager's system prompt as a callable function:

```python
def web_search_agent(task: str, additional_args: dict[str, Any]) -> str:
    """Runs web searches for you. Give it your query as an argument.

    Args:
        task: Long detailed description of the task.
        additional_args: Dictionary of extra inputs to pass to the managed agent,
            e.g. images, dataframes, or any other contextual data it may need.
    """
```

The manager's LLM sees the managed agent exactly like a tool. From the execution perspective, calling `web_search_agent(task="...")` in the manager's generated code triggers `web_agent.run("...")` internally.
