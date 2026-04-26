# Anthropic Engineering: Scaling Managed Agents

**Source:** https://www.anthropic.com/engineering/managed-agents  
**Retrieved:** 2026-04-12  
**Author/Org:** Anthropic Engineering  
**Note:** This article is about Anthropic's hosted managed agent service infrastructure, not about multi-agent orchestration patterns. It addresses how to architect a production system that runs LLM agents at scale by decoupling brain (LLM), harness (orchestration loop), and sandbox (execution environment).

---

## Core Problem

Harnesses encode assumptions about AI capabilities that become outdated as models improve. When all components (session, orchestration loop, sandbox) run in a single container, the system creates "pets" — hand-maintained systems requiring constant nursing.

Specific failures of the coupled approach:
- Container failures meant permanent session loss
- Debugging was impossible without accessing user data
- Network assumptions limited customer flexibility
- Every brain required dedicated container provisioning overhead

---

## The "Brain from Hands" Architecture

The solution separates three virtualized components:

### 1. Session
An append-only log of all events, persisted outside any single container. Sessions serve as external context objects. The `getEvents()` interface lets harnesses interrogate event streams flexibly — rewinding, slicing, or reorganizing for cache optimization — without irreversible compaction decisions.

### 2. Harness
The orchestration loop that calls Claude and routes tool outputs. Made stateless and replaceable. It calls containers via `execute(name, input) -> string`, catches failures as tool-call errors, and can recover using `wake(sessionId)` and `getSession(id)` to resume from the last event.

### 3. Sandbox
Execution environment for code and file operations. Containers provision only when needed via tool calls.

This mirrors how operating systems abstracted hardware through processes and files — implementations change while interfaces remain stable.

---

## Security Boundaries

Credentials never reach sandboxes:
- Git tokens authenticate during initialization only
- OAuth tokens live in secure vaults accessed through a proxy
- Claude calls MCP tools without ever handling tokens directly

---

## Performance Impact

Decoupling yielded significant improvements:
- p50 time-to-first-token dropped ~60%
- p95 dropped over 90%
- Containers provision only on demand via tool calls
- Stateless harness enables horizontal scaling

---

## Design Philosophy

**"Many brains, many hands"**: Each brain can connect to multiple execution environments without coupling, and brains can delegate work to each other.

The harness remains agnostic about whether sandboxes are containers, phones, or other systems. The architecture is opinionated about interfaces while remaining unopinionated about implementations.

---

## Key Insight: Interface Stability

The central lesson: define stable interfaces between components (brain, harness, sandbox) and let implementations evolve independently. This is the same principle as OS-level abstraction — the process model didn't change even as hardware changed completely beneath it.

The `execute(name, input) -> string` contract for tool calls is deliberately minimal. Anything that can accept a name and input string and return a result string can be a sandbox. This is what makes the system extensible without rewriting the orchestration layer.
