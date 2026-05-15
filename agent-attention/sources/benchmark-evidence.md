# Benchmark Evidence: Context Window Doesn't Scale for Agents

## Needle-in-Haystack (Retrieval)
- Claude 2.1: 27% accuracy raw → 98% with prompt engineering
- Claude 3 Opus: >99% at 200K tokens
= RETRIEVAL largely solved by scaling

## RULER Benchmark
- Models show >50% performance degradation at ~100K tokens
- Tasks: retrieval, tracing, multi-hop reasoning

## HELMET Benchmark
- Long-context in-context learning, QA, summarization at 128K
- Performance struggles with longer sequences

## SWE-bench (Coding Agent Tasks)
- Agentic workflow: ~30% resolve rate
- Single-shot long-context: ~7% resolve rate
= Agents with structured memory beat brute-force context

## VideoWebArena (Multimodal Agents)
- 5-10.3% performance drop in skill retention tasks with longer context
- Factual retention: 13.3% success vs. 73.9% for humans

## Key Pattern
Retrieval tasks → solved by larger windows
Reasoning/agent tasks → NOT solved; often made worse

---
# Goal Drift Study (arXiv:2505.02709, May 2025)
Stock trading simulation with portfolio managers

## Mechanism
3 root causes of goal drift:
1. Contextual Attenuation — reduced attention to initial objectives as context grows
2. Pattern-Matching Bias — recent-context patterns dominate over original goal alignment  
3. Local Optimization — step-by-step locally optimal decisions diverge from global objective

## Result
ALL evaluated agents showed goal drift during extended operations.
Best performer: scaffolded Claude 3.5 Sonnet maintained adherence for 100,000+ tokens.
"Drift through inaction" more prevalent than active deviation.

---
# GraphReader (arXiv:2406.14550, June 2024)
## Key Result
4K-context agent OUTPERFORMS GPT-4-128K on LV-Eval benchmark (16K-256K token tasks)
Method: Structures long text as knowledge graph, agent traverses nodes
Why it works: No "lost in the middle" problem — graph traversal targets relevant nodes
Implication: Context management > context capacity

---
# Chain-of-Agents (arXiv:2406.02818, 2024, Google Research)
## Key Results
+10.22% on MuSiQue vs RAG and full-context baselines
+13.30% on NarrativeQA
## Architecture
Worker agents: process short chunks (avoids attention dilution)
Manager agent: synthesizes all worker outputs
Complexity: O(nk) vs O(n²) for full attention

---
# Quadratic Cost Real-World Impact (2024)
34B model at 100K token context: ~5 concurrent users
34B model at 4K token context: ~100+ concurrent users
KV cache: 22.8 GB at 100K vs 0.91 GB at 4K

---
# 2025 新数据

## NoLiMa Benchmark（arXiv:2502.05167，Adobe Research，2025）
非词汇匹配检索（不依赖字面匹配，需要语义推理）

| 模型 | 1K tokens | 8K tokens | 32K tokens |
|------|-----------|-----------|------------|
| GPT-4o | 99.3% | 89.2% | 69.7% |
| Claude 3.5 Sonnet | 85.4% | 61.7% | 29.8% |
| Llama-3.3-70B | 91.0% | 62.7% | 43.2% |

- 13 个被测模型中，11 个在 32K token 时掉到短上下文基准的 50% 以下
- DeepSeek-R1（推理优化模型）在 32K 时也低于 50%
- 关键发现：词汇相似度高的任务（needle 和 haystack 字面相关），扩容有效
  词汇相似度低的任务（需要语义关联），扩容完全没用

## Chroma Context Rot 报告（2025）
18 款主流模型全部测试，包括 GPT-4.1、Claude 4、Gemini 2.5、Qwen3

- 10K tokens = 新的失效临界点
- 超过 10K 后，大部分模型准确率跌到约 50%，无论初始能力多强
- 非线性下降：Claude Sonnet 4 在 1K token 后出现悬崖式下降（90% → 60%）
- 干扰项效应极强：加入 1 个干扰项 → 准确率下降 10-15%；加入 4 个 → 下降 30-50%
- 结构化文本（逻辑连贯）比随机打乱的文本表现更差（模型被连贯叙事带偏）
- 结论：百万 token 上下文窗口在现实任务里存在"脏秘密"
来源：https://research.trychroma.com/context-rot

## WebAgent 长上下文评测（arXiv:2512.04307，2025）
测试 Claude-3.7、GPT-4.1、Llama 4、o4-mini

- 短上下文基准成功率：40-50%
- 上下文超过 25K-150K tokens 后：成功率 <10%
- GPT-4.1 进入循环的频率：44.29%（任务失败的首要原因）
- 失败模式：Agent 陷入循环、失去任务连贯性

## SWE-bench 2025 现状
- Claude Opus 4.5：SWE-bench Verified 解决率 80.3%（结构化 Agent 工作流）
- Claude 4.6 (Sonnet)：87.6%（厂商报告）
- 单次长上下文模式：依然约 7%（2023 数据，近年未见明显改善）
- 差距依然：结构化 Agent vs 暴力长上下文 = 80% vs 7%

## MemAgent（arXiv:2507.02259，字节跳动 + 清华，2025）
RL 训练的内存管理 Agent

- RULER 基准 512K 上下文：>95% 准确率
- 外推能力：从 8K token 训练 → 350 万 token 任务，性能下降 <5%
- 复杂度：O(N)，彻底绕开 O(n²)
- 方法：把文档切块，用 RL 训练模型学会「什么值得写进记忆缓冲区」
- 意义：RL 学会了注意力管理，而不是靠更大的窗口

## SWE-Pruner（arXiv:2601.16746，上交大 + 字节，2026）
编程 Agent 的上下文剪枝框架

- Token 使用量减少 23-54%，任务成功率下降 <1%
- 关键发现：SWE-bench 里 76.1% 的 token 消耗来自文件操作（cat/grep 读文件）
- 在失败→成功的场景里，可节省 83.3% 的 token
- 方法：根据当前任务目标动态剪枝上下文，保留语义相关内容
- 意义：精确管理 > 暴力保留
