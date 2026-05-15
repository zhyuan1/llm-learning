# Solutions: What Actually Works

## 1. MemGPT / Tiered Memory (arXiv:2310.08560, UC Berkeley)
Principle: Separate storage from activation
- Context window = RAM (working memory)
- External storage = disk (archival memory)
- Agent autonomously decides what to page in/out
Result: Processes docs 10x longer than baseline context, 41% higher conversational consistency

## 2. GraphReader (arXiv:2406.14550, June 2024)
Principle: Structure over brute force
- 4K context window
- Long doc → knowledge graph → agent traverses graph
Result: OUTPERFORMS GPT-4-128K on 16K-256K token tasks (LV-Eval benchmark)
Key insight: Graph traversal has no "lost in the middle" problem

## 3. Chain-of-Agents (arXiv:2406.02818, Google Research 2024)
Principle: Divide and conquer
- Worker agents: each processes short chunk (no attention dilution)
- Manager agent: synthesizes outputs
- Complexity: O(nk) instead of O(n²)
Results: +10.22% MuSiQue, +13.30% NarrativeQA vs RAG and full-context baselines

## 4. Context Engineering (Anthropic, practitioners 2025)
Principle: Discipline over capacity
Techniques:
a. Compaction — summarize near limits, keep decisions, discard redundant outputs
b. Structured Note-Taking — NOTES.md for persistent tracking across resets
c. Sub-Agent Architecture — specialized agents, condensed summaries to coordinator
d. Just-in-time retrieval — load data only when needed, not pre-stuffed

## 5. Sparse Attention / FlashAttention (Engineering 2024)
Principle: Reduce the O(n²) cost architecturally
- FlashAttention-2: O(N²) memory → O(N) via GPU memory hierarchy tricks
- Dynamic Sparse (MInference): 10× prefill speedup on A100
- DeepSeek-V3 DSA: 65% GPU memory reduction for 16K token sequences
- LongGen hybrid: 62% KV cache reduction
Limitation: Engineering fix, doesn't solve the attention-quality problem, only the cost problem

## 6. State-Space Models (Mamba, etc.)
Principle: Replace attention with linear-time recurrence
- O(n) instead of O(n²)
- Different failure mode: exponential decay of distant information
- Not yet proven for complex multi-hop reasoning at scale

## Key Pattern Across All Solutions
NONE of them work by "giving more context"
ALL of them work by being smarter about WHAT goes into context at any moment
→ The problem is always attention management, not storage capacity
