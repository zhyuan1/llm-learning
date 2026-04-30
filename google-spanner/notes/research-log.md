# Research Log — Google Spanner

- Mode: Deep Research
- Date: 2026-04-30
- Question: 分析 Google Spanner 全球分布式数据库：出现的背景、解决的问题、如何解决、引入了哪些新问题、带来了哪些启发。
- Primary-source strategy:
  1. 先建立谱系：Bigtable → Megastore → Spanner → F1 / SQL System。
  2. 再补一致性与工程实现：TrueTime / External Consistency / Life of Reads & Writes。
  3. 最后归纳 trade-off：延迟、网络假设、硬件依赖、跨分片事务成本、schema/data modeling 约束。
