# Managed Agents Architecture: How Anthropic Separated Brain from Hands

**Status:** Draft (Phase 4 complete, Phase 5 refined)  
**Date:** 2026-04-12  
**Sources:** Anthropic Engineering blog, Juejin technical community, Sohu tech coverage

---

## 1. The problem: pet-style architecture doesn't scale

Early agent systems packed everything into one container: LLM inference, orchestration loop, code execution, file operations, session state. That container was a "pet" — it needed rescue when broken, couldn't be swapped, and debugging required access to user data.

Specific failures:
- Container crash = permanent session loss
- Debugging limited to WebSocket event streams, with no way to locate failure source
- Customers with resources in their own VPC had to set up network peering
- Each agent instance required a dedicated container, wasting resources

The deeper issue: **the harness encodes assumptions about what models cannot do, and these assumptions become obsolete as models improve.** When all components are coupled, upgrading any one of them affects everything else.

---

## 2. Core idea: virtualize the agent's components

Anthropic's solution borrows from operating system design. Operating systems abstracted hardware through processes and files — implementations changed while interfaces stayed stable. Managed Agents applies the same principle: split the agent into three independently replaceable components connected by stable interfaces.

| Component | Role | Key property |
|-----------|------|-------------|
| **Session** | Persistent memory | Append-only event log, stored outside containers |
| **Harness** | Brain's control loop | Stateless, calls Claude and routes tool outputs |
| **Sandbox** | Hands' execution environment | Isolated code execution container, provisioned on demand |

Design principle: **interfaces are more durable than implementations.** As long as the interface contracts between Session, Harness, and Sandbox hold, each layer can evolve independently.

---

## 3. Session: the append-only event log

The Session is a durable record of all events, stored outside any container (Postgres, SQLite, etc.). It is not the context window — the context window has a length limit and gets consumed as conversation progresses. Session is complete external memory.

### Writing

The Harness writes events via `emitEvent(id, event)`. Every tool call, every Claude response, every error gets recorded. The event stream is append-only, never modified.

### Reading

The Harness reads events via the `getEvents()` interface, which supports flexible retrieval patterns:
- Resume from last read position
- Rewind to before a specific moment
- Re-read prior context
- Slice by time range

This means the Harness controls how context is organized — including rearranging events for higher prompt cache hit rates. This flexibility matters because future models may require entirely different context engineering strategies.

### Why not just compress the context window

Compression is irreversible. Once you summarize 10 events into 1, the original details are gone. Session keeps the complete event stream externally, letting the Harness select on demand without making irreversible decisions.

---

## 4. Harness: the stateless orchestrator

The Harness is the control loop that calls Claude. It reads events from Session, constructs the prompt, sends it to Claude, receives Claude's output, routes tool calls to Sandbox, and writes results back to Session.

Key design: **the Harness holds no state.** All state lives in Session.

### Crash recovery

When a Harness crashes, spin up a new one:
1. `wake(sessionId)` to resume the session
2. `getSession(id)` to retrieve the event log
3. Continue from the last event

This is "cattle, not pets" — the Harness is replaceable infrastructure. If it goes down, start a new one. The user doesn't notice.

### The tool call contract

The Harness calls Sandbox through a minimal interface:

```
execute(name, input) → string
```

`name` is the tool name, `input` is the input string, returns a result string. The Harness doesn't care whether the Sandbox is a container, a phone, or a Pokemon emulator. Anything that accepts a name and input string and returns a result string can be a Sandbox.

If the tool call fails, the Harness wraps the failure as a tool-call error and returns it to Claude. Claude can decide to retry or try a different approach.

---

## 5. Sandbox: execution environments provisioned on demand

The Sandbox is the isolated environment for code execution and file operations. Containers are created on demand via `provision({resources})`, only when a tool call needs them.

### Isolation technologies

Two mainstream approaches in the industry:

**Firecracker microVM**: Hardware-level virtualization with independent guest OS kernels. Strongest isolation (5/5), cold startup 125-150ms. Used by AWS Lambda and Manus.

**gVisor containers**: User-space lightweight kernel where the Sentry runtime intercepts syscalls. Isolation 4/5 (requires additional hardening), cold startup <50ms.

### Multi-tenant isolation

Core principle: **never share instances across users.**

Violations enable: data leakage via `ls -R /`, resource starvation from one user's infinite loops, audit trail corruption.

Isolation methods:
- Each user/task gets a dedicated sandbox instance
- Control plane injects user-level config prefixes (`memory_prefix: f'user-{user_id}/assistant-{assistant_id}/'`)
- Namespace isolation at storage and memory layers

---

## 6. Security boundary: credentials never enter the sandbox

This is one of the architecture's most important security decisions. Claude's generated code runs inside the sandbox, and the sandbox could be exploited via prompt injection. If credentials are in the sandbox, attackers get them.

Anthropic's approach:

### Git authentication
Repository access tokens are used during initialization to clone repos and wired into local git remote config. Subsequent `push` and `pull` operations go through the local remote. The agent never touches the token directly.

### OAuth authentication
OAuth tokens live in a secure vault. When Claude calls external tools via MCP, the request goes through a dedicated proxy. The proxy takes a session-associated token, fetches real credentials from the vault, and makes the external service call. Claude never handles OAuth tokens.

### Tool Proxy pattern (general-purpose)
1. Sandbox-side tools hold only ephemeral `sandbox_token` (short-lived JWT or Redis key)
2. Control plane proxy validates the token, then calls external APIs with real credentials
3. Even if the container is compromised, API keys don't leak

---

## 7. Performance gains

Decoupling yielded significant latency improvements for a straightforward reason: previously, inference couldn't start until the container finished booting. Now the Harness begins inference as soon as it has the Session ID, while the sandbox provisions in the background.

| Metric | Improvement |
|--------|------------|
| p50 time-to-first-token | ~60% reduction |
| p95 time-to-first-token | >90% reduction |

On-demand container creation also means no pre-allocated containers per agent instance. The stateless Harness scales horizontally — multiple Harness instances can process the same Session.

---

## 8. Many brains, many hands

After decoupling, the system gains combinatorial flexibility:

- **One brain, many hands**: A single Harness can call multiple sandboxes (different language runtimes, different external services) with no coupling between them.
- **Brains delegate to brains**: Multiple Harnesses can collaborate. One Harness can pass subtasks to another, which uses its own Sandbox to execute. Shared Sessions enable parallel task handling.
- **Hands pass between brains**: No hand is bound to a specific brain.

The Harness is agnostic about whether the Sandbox is a container, a phone, or another system. This agnosticism is the source of the system's extensibility — new execution environments plug in without rewriting the orchestration layer.

---

## 9. Mental model

The core of Managed Agents architecture is an OS-level abstraction: virtualize the agent's components and isolate them through stable interfaces.

Three components, each independent:
- **Session**: Append-only event log. Persistent external memory, outlasts context windows.
- **Harness**: Stateless orchestration loop. Crash-recoverable, horizontally scalable.
- **Sandbox**: On-demand isolated execution environment. Credentials stay outside.

One minimal tool-call contract connects them: `execute(name, input) → string`.

This architecture lets each layer evolve independently. Model gets stronger — update Harness assumptions without touching Sandbox. Need a new execution environment — add a new Sandbox implementation without touching Harness. Need a different context strategy — change how `getEvents()` is called without touching Session's storage format.

Interfaces are more durable than implementations. That's the single most important sentence in the entire design.

---

## Sources

- [Anthropic Engineering: Managed Agents](https://www.anthropic.com/engineering/managed-agents) — Primary source
- [Juejin: Agent Sandbox Architecture Deep Dive](https://article.juejin.cn/post/7618805017437650987) — Sandbox isolation comparison
- [Sohu: Anthropic's Harness Philosophy](https://www.sohu.com/a/1008149966_122189055) — Chinese tech coverage
- [CNBlogs: Managed Agents Full Analysis](https://www.cnblogs.com/wind-xwj/articles/19843533) — Chinese technical writeup
