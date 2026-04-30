# Google Spanner Primary Sources

## Core lineage
- Bigtable: A Distributed Storage System for Structured Data
  - URL: https://research.google/pubs/bigtable-a-distributed-storage-system-for-structured-data/
  - Local PDF: `primary/bigtable-osdi-2006.pdf`
  - Why: Spanner 之前的可扩展存储起点，说明了缺少跨行事务和全局一致性的问题边界。
- Megastore: Providing Scalable, Highly Available Storage for Interactive Services
  - URL: https://www.cidrdb.org/cidr2011/Papers/CIDR11_Paper32.pdf
  - Local PDF: `primary/megastore-cidr-2011.pdf`
  - Why: Spanner 的直接前身，展示 Paxos 复制和 entity group 的能力与瓶颈。
- Spanner: Google's Globally-Distributed Database
  - URL: https://research.google/pubs/spanner-googles-globally-distributed-database-2/
  - Local PDF: `primary/spanner-osdi-2012.pdf`
  - Why: Spanner 核心设计论文。

## On top of Spanner
- F1: A Distributed SQL Database That Scales
  - URL: https://research.google/pubs/pub41344/
  - Local PDF: `primary/f1-vldb-2013.pdf`
  - Why: 说明真实业务如何在 Spanner 之上承受同步复制延迟并构建 SQL 系统。
- Spanner: Becoming a SQL System
  - URL: https://research.google/pubs/spanner-becoming-a-sql-system/
  - Local PDF: `primary/spanner-sql-system-sigmod-2017.pdf`
  - Why: 展示 Spanner 从底层分布式事务系统演进为完整 SQL 系统的关键设计。

## Consistency and operations
- Spanner, TrueTime and the CAP Theorem
  - URL: https://research.google/pubs/spanner-truetime-and-the-cap-theorem/
  - Local snapshot: `primary/spanner-truetime-cap-theorem.md`
  - Why: 解释 Spanner 为什么没有“绕过” CAP，而是在 Google 网络假设下取得更像 CA 的体验。
- Spanner: TrueTime and External Consistency
  - URL: https://docs.cloud.google.com/spanner/docs/true-time-external-consistency
  - Local snapshot: `primary/spanner-true-time-external-consistency.md`
  - Why: 官方解释 TrueTime、MVCC、external consistency 与强读语义。
- Life of Spanner Reads & Writes
  - URL: https://docs.cloud.google.com/spanner/docs/whitepapers/life-of-reads-and-writes
  - Local snapshot: `primary/spanner-life-of-reads-and-writes.md`
  - Why: 以工程流程视角拆开单分片/多分片读写路径，补足论文中的实现细节。
- Spanner Whitepapers Index
  - URL: https://docs.cloud.google.com/spanner/docs/whitepapers
  - Local snapshot: `primary/spanner-whitepapers-index.md`
  - Why: 官方索引页，方便后续扩展到 query lifecycle、schema design、failover 等主题。
