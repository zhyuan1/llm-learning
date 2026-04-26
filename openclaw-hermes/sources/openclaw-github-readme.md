# OpenClaw — Personal AI Assistant

**Source:** https://github.com/openclaw/openclaw  
**Retrieved:** 2026-04-26  
**Author/Org:** openclaw (community, MIT)

---

## Core Architecture

Gateway is the control plane, not the executor. Three layers:

```
Chat Apps / Plugins
      ↓
Gateway (Node.js, WebSocket control plane)
— manages sessions, presence, config, Cron, Webhooks, Control UI, Canvas
      ↓
Pi Agent (RPC subprocess — actual AI inference)
      ↓
Nodes (iOS / Android / macOS devices)
```

The Gateway does not execute tools directly. Pi Agent handles AI inference via RPC. Nodes extend Gateway capabilities to native hardware.

---

## Technology Stack

- **Runtime**: Node.js 24 (recommended), 22.14+ minimum
- **Language**: TypeScript (~90%)
- **Build**: pnpm (preferred), Vitest for testing, Oxlint + Prettier
- **Deployment**: Docker, Fly.io, Render, systemd/launchd daemon

TypeScript is not arbitrary — Voice Wake needs system APIs, iOS/Android Nodes need native bridges, Canvas needs WebSocket push rendering. These requirements mandate a distributed control plane.

---

## Key Features

**Multi-channel (26+):** WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage, Microsoft Teams, Matrix, Feishu, LINE, WeChat, QQ, and more. Plugin-based channels extend this further.

**Multi-agent routing:** Route inbound channels/accounts to isolated agents with separate workspaces. Cross-agent communication via `sessions_send`, `sessions_spawn`, `sessions_list`.

**Native companion apps:**
- macOS: Menu bar app (Voice Wake, PTT, Talk Mode overlay, Canvas, debug tools)
- iOS Node: Canvas, Voice Wake, Talk Mode, Camera, Screen capture, Bonjour pairing
- Android Node: Connect/Chat/Voice tabs, Canvas, Camera, Screen capture

**Live Canvas:** Agent-driven visual workspace using A2UI, rendered via WebSocket.

---

## Security Model

**Trust model:** "Personal assistant trust model" — one trusted operator, possibly multiple agents. Explicitly not a shared multi-tenant bus.

**DM policy:** Default `dmPolicy="pairing"` — unknown senders get a short pairing code challenge. `"open"` requires explicit opt-in.

**Sandbox (three levels):**

| Mode | Behavior |
|------|----------|
| `off` | Default — tools run on host |
| `non-main` | Non-main sessions run in Docker sandbox |
| `all` | All sessions sandboxed |

Default sandbox: allows `bash`, `process`, `read`, `write`; denies `browser`, `canvas`, `nodes`, `cron`.

**Additional hardening:** `workspaceOnly` options for file/patch operations; `openclaw security audit --deep` with `--fix` auto-remediation; full `SECURITY.md` with threat model and out-of-scope definitions.

---

## Skills System

**Three skill types:**
1. Bundled — shipped with core
2. Managed — installed/updated via ClawHub (`clawhub.ai`)
3. Workspace — local, project-specific

Skills invoked as slash commands. Agent behavior configured via context files: `AGENTS.md` (agent definitions), `SOUL.md` (personality), `TOOLS.md` (available tools).

---

## Engineering Decisions

- Local-first: privacy and control over cloud dependency
- Single gateway process: manages all platform adapters in one process
- Plugin trust: installing a plugin grants host-level trust — same as local code
- TypeScript everywhere: required for native app bridges and type safety across the stack
- Workspace isolation: per-agent workspaces for multi-agent scenarios
