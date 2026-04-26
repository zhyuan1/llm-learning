# Hermes Agent — The Self-Improving AI Agent

**Source:** https://github.com/NousResearch/hermes-agent  
**Retrieved:** 2026-04-26  
**Author/Org:** Nous Research (MIT)

---

## Core Architecture

Single Python process + SQLite file. The entry point is `hermes`, which starts `AIAgent` in `run_agent.py` (~3600 lines) — the complete conversation loop.

```
hermes CLI / hermes gateway
      ↓
AIAgent (run_agent.py, ~3600 lines)
— synchronous main loop
— tool dispatch via ToolRegistry
— session persistence via SessionDB (SQLite + FTS5)
```

**Why synchronous (not async):** The bottleneck is LLM API latency, not I/O concurrency. Synchronous code is easier to reason about, debug, and maintain. Parallelism is handled explicitly via `ThreadPoolExecutor`.

**Five-layer architecture:**

| Layer | Key Files |
|-------|-----------|
| Entry & Orchestration | `cli.py` (HermesCLI), `gateway/run.py` (GatewayRunner) |
| Agent Core | `run_agent.py` (AIAgent), `agent/prompt_builder.py`, `agent/context_compressor.py` |
| Tool & Registry | `tools/registry.py`, `model_tools.py`, `toolsets.py` |
| State & Persistence | `hermes_state.py` (SessionDB), `tools/memory_tool.py` (MemoryStore) |
| Platform Adapters | `gateway/platforms/`, `environments/` |

---

## Closed Learning Loop

The defining feature — a self-reinforcing capability cycle:

```
Task Execution → Experience Capture → Skill Creation
      ↓
Skill Refinement (during use) → Memory Persistence
      ↓
Cross-Session Recall (FTS5 + LLM summarization)
      ↓
User Modeling (Honcho dialectic profiling)
```

Skills are procedural memory created autonomously by the agent, stored as Markdown in `~/.hermes/skills/`, auto-invoked for similar future tasks, and self-improving during use.

---

## Memory System (Bounded & Curated)

| Store | Limit | Purpose |
|-------|-------|---------|
| `MEMORY.md` | 2200 chars | Environment facts, project conventions |
| `USER.md` | 1375 chars | User preferences, workflow habits |

Character limits (not token) — model-agnostic. When near the limit, the agent must replace or delete existing entries before adding new ones. This constraint forces the agent to prioritize what's worth keeping.

**Frozen snapshot pattern:** Memory is loaded at session start and injected into the system prompt. Mid-session writes persist to disk but don't alter the current session's system prompt — preserves Anthropic prefix cache validity for the entire session.

**Atomic writes:** `os.replace()` for write consistency; `fcntl.flock` for concurrent write protection.

---

## RL Data Flywheel

Hermes includes `tinker-atropos` for RL training integration:

```
Large model (Claude) completes tasks via Hermes
      ↓
Export full trajectories as JSONL
(system prompt, user messages, model reasoning, tool calls + results)
      ↓
Fine-tune smaller model on trajectories
      ↓
Reduce API costs, maintain performance
```

`trajectory_compressor.py`: compresses completed trajectories to token budget while preserving training signal — protect first + last N turns, compress middle, replace with summary message.

---

## Terminal Backends (6)

| Backend | Use Case |
|---------|----------|
| Local | Direct machine execution |
| Docker | Containerized isolation |
| SSH | Remote server execution |
| Daytona | Serverless with persistence, hibernation when idle |
| Singularity | HPC container runtime |
| Modal | Serverless, near-zero idle cost |

---

## OpenClaw Migration

```bash
hermes claw migrate
```

Imports from OpenClaw: SOUL.md, MEMORY.md, user-created skills, command allowlist, messaging settings, API keys, TTS assets, workspace instructions.

---

## Model Agnostic

200+ models via single `hermes model` command: Nous Portal, OpenRouter, NVIDIA NIM, OpenAI, Anthropic, Kimi/Moonshot, MiniMax, Hugging Face, any custom OpenAI-compatible endpoint.
