# smolagents: Building Good Agents

**Source:** https://huggingface.co/docs/smolagents/main/en/tutorials/building_good_agents  
**Retrieved:** 2026-04-12  
**Author/Org:** HuggingFace (smolagents team)

---

## Core principle: simplify the workflow as much as you can

Giving an LLM agency introduces risk of errors. Well-programmed agentic systems have error logging and retry mechanisms so the LLM can self-correct. But to reduce the risk of LLM error, simplify the workflow.

**Main guideline: reduce the number of LLM calls as much as you can.**

Takeaways:
- Whenever possible, group 2 tools into one (e.g., instead of separate "travel distance API" and "weather API" calls, build one unified `return_spot_information` tool)
- Whenever possible, logic should be based on deterministic functions rather than agentic decisions

---

## Improve the information flow to the LLM engine

The LLM engine is like an intelligent robot in a room with the only communication being notes passed under a door. It won't know anything that happened unless you explicitly put it in the prompt.

### Bad tool example (poor information flow)

```python
@tool
def get_weather_api(location: str, date_time: str) -> str:
    """
    Returns the weather report.

    Args:
        location: the name of the place that you want the weather for.
        date_time: the date and time for which you want the report.
    """
    lon, lat = convert_location_to_coordinates(location)
    date_time = datetime.strptime(date_time)
    return str(get_weather_report_at_coordinates((lon, lat), date_time))
```

Problems: no precision on `date_time` format, no detail on location format, no error logging, hard-to-understand output format.

### Good tool example (rich information flow)

```python
@tool
def get_weather_api(location: str, date_time: str) -> str:
    """
    Returns the weather report.

    Args:
        location: the name of the place that you want the weather for.
            Should be a place name, followed by possibly a city name, then a country,
            like "Anchor Point, Taghazout, Morocco".
        date_time: the date and time for which you want the report,
            formatted as '%m/%d/%y %H:%M:%S'.
    """
    lon, lat = convert_location_to_coordinates(location)
    try:
        date_time = datetime.strptime(date_time)
    except Exception as e:
        raise ValueError(
            "Conversion of `date_time` to datetime format failed, "
            "make sure to provide a string in format '%m/%d/%y %H:%M:%S'. "
            "Full trace:" + str(e)
        )
    temperature_celsius, risk_of_rain, wave_height = get_weather_report_at_coordinates(
        (lon, lat), date_time
    )
    return (
        f"Weather report for {location}, {date_time}: "
        f"Temperature will be {temperature_celsius}°C, "
        f"risk of rain is {risk_of_rain*100:.0f}%, "
        f"wave height is {wave_height}m."
    )
```

**Test question:** "How easy would it be for me, if I was dumb and using this tool for the first time, to program with this tool and correct my own errors?"

---

## Debugging your agent

### 1. Use a stronger LLM

Many agent errors are LLM reasoning failures, not system bugs. Upgrade the model first.

### 2. Provide more information or specific instructions

- Always-on instruction (like a system prompt): pass as string under argument `instructions` upon agent initialization. Note: instructions are *appended* to the system prompt, not replacing it.
- Task-specific instruction: add all details to the task itself. Tasks can be very long.
- Tool-specific instruction: include in the `description` attribute of the tool.

### 3. Change the prompt templates (generally not advised)

System prompt template can be overridden via `system_prompt` parameter. Available placeholders:
- Tool descriptions: `{%- for tool in tools.values() %}`
- Managed agent descriptions: `{%- if managed_agents and managed_agents.values() | list %}`
- Authorized imports (CodeAgent only): `{{authorized_imports}}`

Simpler approach: pass `instructions` upon agent initialization.

### 4. Extra planning

Enable a supplementary planning step where the agent regularly updates a fact list and reflects on next steps (no tool call, just reasoning):

```python
agent = CodeAgent(
    tools=[search_tool, image_generation_tool],
    model=InferenceClientModel(model_id="Qwen/Qwen2.5-72B-Instruct"),
    planning_interval=3  # planning step every 3 action steps
)
```

---

## Key insight: tools are the interface

Each tool needs:
- A clear, descriptive name
- Precise type hints on inputs and output
- A description with an `Args:` section (each argument described)
- Explicit error logging inside `forward` method

These attributes are baked into the agent's system prompt at initialization. Poor tool descriptions directly degrade agent performance.
