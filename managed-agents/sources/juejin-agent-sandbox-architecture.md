# Anthropic Managed Agents: Agent Sandbox Architecture Patterns

**Source:** https://article.juejin.cn/post/7618805017437650987  
**Retrieved:** 2026-04-12  
**Author/Org:** 掘金技术社区  
**Note:** Comprehensive comparison of sandbox isolation strategies for production agent systems. Covers Firecracker microVMs, gVisor containers, multi-tenant isolation, Tool Proxy pattern, and framework boundary principles.

---

## Two Core Sandbox Patterns

### Pattern 1: Agent-in-Sandbox
Entire Agent (including LLM reasoning loop) runs inside the sandbox. Local execution eliminates network latency between reasoning and action.

- Firecracker microVMs or persistent containers
- Complete OS access with root privileges and systemd
- Examples: Manus, Happycapy, Devin cloud version
- Trade-off: higher resource cost per instance, maximum autonomy

### Pattern 2: Code Interpreter Sandbox
LLM and Agent orchestration run on servers; only code/commands sent to sandbox for execution.

- Server-side Agent orchestration (control plane)
- Sandbox operates as remote execution tool via network calls
- Examples: OpenPerplexity, ChatGPT Code Interpreter, Claude Desktop
- Trade-off: network round-trips per action, but cheaper multi-tenant

---

## Container & Isolation Technologies

### Firecracker microVMs
- Hardware-level virtualization, independent guest kernels
- Cold startup: 125-150ms
- Isolation strength: 5/5
- Used by: AWS Lambda, Manus

### Persistent OCI Containers with gVisor
- User-space lightweight kernel (Sentry runtime intercepts syscalls)
- Cold startup: <50ms
- Isolation strength: 4/5 (requires additional hardening)
- Namespace + cgroup isolation

---

## Multi-Tenant Isolation

**Core principle: strictly prohibit multi-user instance sharing.**

Violations enable: data leakage via `ls -R /`, resource starvation, audit trail corruption.

Isolation strategies:
1. Physical separation: each user/task gets a dedicated sandbox instance
2. Configuration injection: control plane injects user-specific prefixes (`memory_prefix: f'user-{user_id}/assistant-{assistant_id}/'`)
3. Namespace isolation at storage and memory layer

---

## Tool Proxy Pattern (API Key Security)

Prevents credential exposure in Agent-in-Sandbox mode:

1. Sandbox-side tools hold only ephemeral `sandbox_token` (short-lived JWT or Redis key)
2. Control plane proxy validates token, then calls external API with real credentials
3. API keys never enter sandbox environment, even if container is compromised

---

## High-Concurrency Architecture

**Agent-in-Sandbox**: Horizontal scaling by provisioning independent VMs/containers. Central service only distributes tokens — lightweight.

**Orchestrator-heavy models** (e.g., Perplexity): Central orchestrator maintains thousands of HTTP connections. Concurrency pressure is severe — Perplexity frequently throttles unlimited tiers for this reason.

---

## Framework Boundary Principles

### Framework should NOT handle:
- user_id and tenant identification
- Multi-tenant resource allocation
- Deployment mode detection (local vs SaaS)
- Global rate limiting or quota management

### Framework SHOULD handle:
- Single-sandbox resource management (memory limits, temp cleanup)
- Performance optimization (lazy imports, execution caching)
- Self-protection (timeout enforcement, process reaping)
- Lifecycle hooks for control plane integration

---

## Five Production Presets

| Preset | Key Config |
|--------|-----------|
| memory-optimized | Auto GC, 400MB limit, restricted context windows |
| production-log | Structured JSON output, step-level metrics |
| long-term-memory | S3 backend, semantic indexing |
| safe-mode | 50-step max, 512MB threshold, dead-loop detection |
| local-light | In-memory backend, minimal dependencies |
