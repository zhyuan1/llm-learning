# Hermes Agent Skills Hub

**Source:** https://hermes-agent.nousresearch.com/docs/skills/  
**Retrieved:** 2026-04-26  
**Author/Org:** Nous Research

---

## Registry Statistics

- **Total:** 654 skills across 4 registries
- **Categories:** 16
- **Open standard:** Compatible with `agentskills.io`

| Type | Count |
|------|-------|
| Built-in | 74 |
| Optional | 59 |
| Community | 521 |
| Anthropic | 16 |
| LobeHub | 505 |

---

## Skill Design Principles

**Automatic activation:** Skills load based on user intent — explicit mentions, keyword matching, file type detection, task type, domain keywords. No manual invocation needed.

**Fallback mechanisms:** Most skills provide multiple implementation paths. Example: GitHub skills use `gh` CLI first, fall back to GitHub REST API via curl.

**Composition patterns:**
- Delegation: skills delegate to specialized CLI agents (`claude-code`, `codex`, `opencode`)
- Integration: OAuth2/API key connection to external services (Google Workspace, Linear, Notion)
- Pipeline: end-to-end production pipelines (manim-video, p5js, ascii-video)

---

## Hermes Skills vs. OpenClaw Skills

| Aspect | Hermes Skills | OpenClaw (ClawHub) |
|--------|--------------|--------------------------|
| Creation | Auto-created by agent + community hub | Human-written, installed manually |
| Storage | `~/.hermes/skills/` as Markdown docs | ClawHub registry + local workspace |
| Invocation | Automatic by context + explicit `/skill-name` | Slash commands |
| Self-improvement | Yes — improves during use | No |
| Open standard | agentskills.io | Proprietary (ClawHub) |

---

## Notable Skills (Selected)

**Software Dev:** `claude-code` (delegate to Claude Code CLI), `codex` (OpenAI Codex CLI), `github-pr-workflow` (full PR lifecycle), `codebase-inspection`

**Creative:** `architecture-diagram` (dark-themed SVGs, semantic color scheme), `manim-video` (3Blue1Brown-style animations), `excalidraw` (hand-drawn diagrams), `popular-web-designs` (54 production design systems: Stripe, Linear, Vercel, Notion)

**MLOps:** `axolotl` (LLM fine-tuning, LoRA/QLoRA/DPO), `evaluating-llms-harness` (60+ benchmarks), `serving-llms-vllm` (PagedAttention, continuous batching), `unsloth` (2-5x faster fine-tuning)

**Productivity:** `google-workspace` (Gmail/Calendar/Drive, OAuth2), `linear` (GraphQL API), `notion`, `powerpoint`

**Research:** `arxiv` (free REST API), `research-paper-writing` (NeurIPS/ICML/ICLR/ACL pipeline)
