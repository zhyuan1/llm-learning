# 一手资料索引

## 核心论文

| 编号 | 标题                                                                            | 来源                                           | 关键贡献                             |
|----|-------------------------------------------------------------------------------|----------------------------------------------|----------------------------------|
| S1 | UltraHorizon: Benchmarking Agent Capabilities in Ultra Long-Horizon Scenarios | arxiv 2509.21766 (2025.09)                   | 定义 in-context locking，8 种错误类型分类  |
| S2 | TravelPlanner: A Benchmark for Real-World Planning with Language Agents       | arxiv 2402.01622 (ICML 2024 Spotlight)       | GPT-4 成功率仅 0.6%，多约束规划失败          |
| S3 | YC-Bench: AI Agent Long-Term Planning Benchmark                               | arxiv 2604.01212 (2026.04)                   | scratchpad 是跨上下文截断的唯一有效机制        |
| S4 | Scaling Long-Horizon LLM Agent via Context-Folding                            | context-folding.github.io                    | 主动上下文管理 vs 被动线性堆积                |
| S5 | ACON: Optimizing Context Compression for Long-horizon LLM Agents              | arxiv 2510.00615 / github.com/microsoft/acon | 26-54% token 压缩，小模型接近大模型表现       |
| S6 | Vending-Bench: A Benchmark for Long-Term Coherence                            | arxiv 2502.15840                             | 失败与上下文窗口占满无直接相关，是连贯性问题           |
| S7 | Context Rot: LLM Performance Degradation                                      | trychroma.com/research/context-rot           | 上下文越长性能下降越不均匀，干扰词影响非线性           |
| S8 | MemAgent                                                                      | arxiv 2507.02259                             | 分段处理 + 覆写策略，8K 训练外推至 3.5M tokens |

## 背景参考

- Why LLM Reasoning Breaks Down in Long-Horizon Planning Tasks — news.skrew.ai
- HiAgent: Hierarchical Working Memory Management (ACL 2025) — aclanthology.org/2025.acl-long.1575/
