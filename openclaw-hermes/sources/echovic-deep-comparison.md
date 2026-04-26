# Hermes Agent 深度架构剖析与 OpenClaw 工程化对比

**Source:** https://www.echovic.com/blog/ai/hermes-agent-deep-architecture-openclaw-comparison  
**Retrieved:** 2026-04-26  
**Author/Org:** echovic.com

---

## Project Background

| 维度 | Hermes Agent | OpenClaw |
|------|--------------|----------|
| 首次发布 | 2025 年中 | 2024 年底 |
| Stars | 118k | 352k |
| 贡献者 | 317 | 3,000+ |
| 主语言 | Python 93.6% | TypeScript 90.3% |
| 定位 | 自进化研究型 Agent | 个人 AI 助手产品 |

---

## Hermes Agent: Five-Layer Architecture

| Layer | Components |
|-------|-----------|
| Entry & Orchestration | `cli.py` (HermesCLI, prompt_toolkit TUI), `gateway/run.py` (GatewayRunner) |
| Agent Core | `run_agent.py` (AIAgent ~3600 lines), `agent/prompt_builder.py`, `agent/context_compressor.py`, `agent/auxiliary_client.py` |
| Tool & Registry | `tools/registry.py` (singleton), `model_tools.py` (triggers tool discovery), `toolsets.py` |
| State & Persistence | `hermes_state.py` (SessionDB, SQLite + FTS5), `tools/memory_tool.py` (MemoryStore), `cron/scheduler.py` |
| Platform Adapters | `gateway/platforms/`, `acp_adapter/`, `environments/` |

**Synchronous main loop rationale:** Agent bottleneck is LLM API latency, not I/O concurrency. Synchronous code is easier to reason about, debug, and maintain. Parallelism via `ThreadPoolExecutor` where needed.

---

## Tool Registry: Three Key Design Patterns

### Pattern 1: Self-Registration

Each tool file calls `registry.register()` on import, declaring schema, handler, toolset membership, `check_fn`, `max_result_size_chars`, `is_async`. Dependency chain: `registry.py` has no dependencies on tool files (no circular imports) → tool files depend on registry → `model_tools.py` imports registry and triggers all tool discovery.

### Pattern 2: Runtime Availability Check

Each tool registers a `check_fn`. Tools requiring an API Key that isn't configured automatically disappear from the tool list — graceful degradation, not exceptions.

### Pattern 3: Dynamic Schema Rebuilding (Anti-Hallucination)

`get_tool_definitions()` dynamically modifies schema content based on currently available tools:
- `execute_code` tool description lists "tools available in sandbox" — dynamically generated
- `browser_navigate` description contains "prefer web_search first" — when `web_search` is unavailable, that reference is automatically removed

**Why this matters:** LLMs call tools mentioned in descriptions even if those tools don't exist. Most agent frameworks produce hallucinations here. Hermes resolves this at the registry layer, not via prompt engineering or runtime error handling.

### Bonus: Argument Type Coercion (`coerce_tool_args()`)

LLMs frequently return numbers as strings (`"42"` instead of `42`). `coerce_tool_args()` compares each parameter against the tool's JSON Schema and attempts safe coercion on type mismatch. Prevents a large class of production tool call failures that are hard to reproduce.

---

## System Prompt: 7-Layer Structure

1. **Identity layer**: Default agent identity or user-defined `SOUL.md`
2. **Platform prompt layer**: Format guidance per platform (WhatsApp/Telegram/Discord/CLI)
3. **Behavior guidance layer**: Memory nudges, session search guidance, skills, tool usage enforcement
4. **Model-specific layer**: `OPENAI_MODEL_EXECUTION_GUIDANCE` / `GOOGLE_MODEL_OPERATIONAL_GUIDANCE` patches
5. **Memory snapshot layer**: Frozen snapshot from MEMORY.md + USER.md
6. **Skills index layer**: Compact index of all installed skills
7. **Context files layer**: AGENTS.md, .hermes.md, project context

**Cache-friendly design:** AGENTS.md explicitly warns against changes that alter past context mid-conversation — preserves Anthropic prefix cache validity for the entire session.

**Injection safety scan:** All externally injected content (memory files, context files, AGENTS.md) scans for prompt injection, role hijacking, data exfiltration patterns, and invisible Unicode characters before entering system prompt.

---

## Memory System: Bounded Curation

| Store | Limit | Purpose |
|-------|-------|---------|
| `MEMORY.md` | 2200 chars | Environment facts, conventions, tool quirks |
| `USER.md` | 1375 chars | User preferences, communication style, workflow habits |

- Character limits (not token) — model-agnostic
- Near the limit: agent must replace/delete existing entries before adding new ones — forces priority management
- **Frozen snapshot pattern**: loaded at session start, injected into system prompt, unchanged for session; mid-session writes persist to disk but don't alter current system prompt → prefix cache valid entire session
- Atomic writes: `os.replace()` (atomic rename); concurrent writes: `fcntl.flock`

---

## Context Compression: Four-Phase Pipeline

1. **Cheap pre-pass (zero LLM calls):** Replace old tool results with placeholders — pure text substitution
2. **Boundary determination:** Protect head (system prompt + first exchange) and tail (token-budget-based recent context, ~20K tokens). Uses token budget, not fixed message count — messages with large tool output consume more protection budget
3. **Structured summarization:** Auxiliary LLM summarizes middle turns using 7-part template (goal, constraints/preferences, progress, key decisions, relevant files, next steps, critical context). Budget = 20% of compressed content, capped at min(5% of model context, 12,000 tokens)
4. **Integrity repair:** Fix orphaned tool_call/tool_result pairs broken by compression — LLM APIs require every tool call to have a matching result

**Iterative update:** Subsequent compressions update the previous summary incrementally rather than re-summarizing from scratch — information from earlier compressions is preserved.

---

## Subagent Delegation

**Isolation principles:**
- Fresh conversation (no parent context inheritance)
- Independent `task_id` (isolated terminal session and file operation cache)
- Restricted toolset (`delegate_task`, `clarify`, `memory` always stripped)

**Concurrency:** Max 3 parallel subagents (`MAX_CONCURRENT_CHILDREN = 3`), `ThreadPoolExecutor`. Max delegation depth: 2 (parent → child → grandchild rejected).

**Global state protection (critical detail):**

```python
def delegate_task(parent_agent, task, ...):
    saved = model_tools._last_resolved_tool_names  # save
    try:
        child = AIAgent(...)
        result = child.run_conversation(...)
    finally:
        model_tools._last_resolved_tool_names = saved  # restore
    return result
```

Prevents child agent's toolset from polluting parent agent's process-global state.

---

## Gateway Architecture

**Single-process multi-platform:** `GatewayRunner` manages all platform adapter lifecycles in one process, shared event loop. Simplifies deployment, but a single platform crash can affect overall stability.

**Command registry (single source of truth):** `COMMAND_REGISTRY` defines all slash commands. CLI, Gateway, Telegram menus, Slack sub-command routing, autocomplete — all consumers derive from this registry. Add one command, all platforms get it automatically.

**SQLite WAL mode:** Supports concurrent readers and single writer — critical for multi-platform gateway scenarios.

---

## Eight-Dimension Engineering Comparison

### Runtime Form

| | Hermes | OpenClaw |
|--|--|--|
| Core | Python process + SQLite | Node.js Gateway + WebSocket |
| Native apps | None | macOS / iOS / Android |
| Idle cost | Near-zero (Daytona/Modal hibernation) | Gateway process always running |
| Essence | Lightweight backend + heavy agent loop | Heavy gateway platform + distributed Node architecture |

### Tool System

| | Hermes | OpenClaw |
|--|--|--|
| Core design | Self-registration + runtime availability + dynamic schema rebuild | Plugin API + npm distribution + sandbox + permission tiers |
| Core question | How to keep tool system consistent with LLM behavior (anti-hallucination) | How to keep tool system consistent with user trust boundaries |
| Sandbox | Optional Docker terminal backend | Three-level sandbox (off/non-main/all) |

### Memory

| | Hermes | OpenClaw |
|--|--|--|
| Design | Bounded curation (2200+1375 chars) + frozen snapshot | Replaceable memory plugin slot |
| Learning loop | Skills auto-created and self-improving | No native loop (human-installed skills) |
| Cross-session search | FTS5 + LLM summarization | Insufficient data |
| Common | Both support Honcho dialectic user modeling | Both support Honcho |

### Task Orchestration

| | Hermes | OpenClaw |
|--|--|--|
| Pattern | Sync loop + subagent delegation + Cron | Multi-agent routing + session tools + Cron + queue mode |
| Multi-agent | Max 3 parallel, depth limit 2 | Full `sessions_send`/`sessions_spawn` primitives |
| Essence | Single agent loop + delegation | Closer to an agent operating system |

### Security

| | Hermes | OpenClaw |
|--|--|--|
| Default posture | Default trust + selective protection | Default secure + selective openness |
| Trust model | Technical users self-manage | "Personal assistant trust model" (documented) |
| Notable | Memory injection scanning, Cron path validation | Full SECURITY.md, tiered sandbox, `security audit --deep` |

---

## Key Code Patterns

### Frozen Snapshot Memory

```
Session start:  memory_snapshot = MemoryStore.load_from_disk()
                system_prompt += memory_snapshot  # frozen injection

Mid-session:    MemoryStore.update()  # immediately persists to disk
                # does NOT change current session's system_prompt

Result:         Prefix cache valid entire session; new memory visible next session
```

### Dynamic Schema Rebuilding

```python
def get_tool_definitions():
    tool_defs = []
    for tool in registry.tools:
        if not tool.check_fn():
            continue  # tool unavailable, skip
        schema = tool.schema.copy()
        # dynamically remove cross-tool references to unavailable tools
        if tool.name == 'browser_navigate' and 'web_search' not in available_tools:
            schema['description'] = schema['description'].replace("prefer web_search", "")
        tool_defs.append(schema)
    return tool_defs
```

---

## Architectural Philosophy

**Hermes:** Deep learning loop route — Agent self-improvement as the core innovation. Synchronous loop keeps reasoning chain controllable; tool registry keeps extension low-friction; RL integration positions Hermes as co-producer of LLM capability, not just consumer.

**OpenClaw:** Full-platform productization route — consumer-grade product experience as the core innovation. 26+ messaging channels, native multi-platform apps, Voice Wake, Canvas, complete security model and operational toolchain. All designed for "non-technical users having a powerful personal AI assistant."
