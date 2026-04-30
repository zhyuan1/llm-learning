# Google Spanner 中文文章提纲

## 备选标题
1. **Google Spanner：全球分布式数据库是怎么被工程化出来的**
2. **从 Bigtable 到 Spanner：Google 如何把全球复制与强一致事务放进同一个系统**
3. **Spanner 的真正价值，不是 TrueTime，而是重新定义了全球数据库的权衡方式**

## 文章中心论点
Spanner 的意义不在于“发明了一个更快的分布式数据库”，而在于它把**全球复制、强一致事务、工程可用性**三件长期互相牵制的目标，变成了一个可以通过系统工程精细权衡的问题。

## 目标读者
- 对分布式系统和数据库有基础概念的工程师
- 想理解 Spanner 为什么重要、而不是只记住 TrueTime 名词的读者
- 想从 Spanner 提炼系统设计启发的人

## 文章结构

### 一、开场：为什么 Spanner 值得单独写一篇
- 先抛出问题：全球多机房部署，为什么通常意味着一致性、延迟和可用性之间的痛苦取舍？
- 点出 Spanner 的代表性：它不是普通数据库产品，而是数据库系统设计史上的分水岭。
- 给出全文主线：
  1. 它为什么出现
  2. 它解决了什么问题
  3. 它怎么做到
  4. 它付出了什么代价
  5. 它给后来系统留下了什么启发
- **主要来源**：
  - `sources/primary/spanner-osdi-2012.pdf`
  - `sources/primary/spanner-truetime-cap-theorem.md`

### 二、Spanner 不是从零开始：它的前史是什么
#### 2.1 Bigtable 解决了什么
- 超大规模结构化数据存储
- 可扩展、灵活、高吞吐
- 但更接近可扩展存储系统，而不是全球强事务数据库
- **主要来源**：
  - `sources/primary/bigtable-osdi-2006.pdf`

#### 2.2 Megastore 往前迈了哪一步
- 在 Bigtable 之上加入跨数据中心复制与 ACID 语义
- 用 Paxos 和更强事务语义弥补纯存储系统的不足
- 但能力边界、事务范围和性能代价都很明显
- **主要来源**：
  - `sources/primary/megastore-cidr-2011.pdf`

#### 2.3 Google 真正缺的是什么
- Google 已经会“把数据存很大”
- 但还不会“在全球范围内同步复制、事务化，并让业务像写传统数据库一样简单”
- 引出 Spanner 的诞生动机
- **主要来源**：
  - `sources/primary/bigtable-osdi-2006.pdf`
  - `sources/primary/megastore-cidr-2011.pdf`
  - `sources/primary/spanner-osdi-2012.pdf`

### 三、Spanner 到底想解决什么问题
- 全球分布数据时，如何仍然支持强一致事务
- 事务顺序如何与外部世界观察到的顺序保持一致
- 如何避免“要么强一致、要么完全不可用”的极端体验
- 为什么这比“做一个跨机房数据库”难得多
- **主要来源**：
  - `sources/primary/spanner-osdi-2012.pdf`
  - `sources/primary/spanner-true-time-external-consistency.md`

### 四、Spanner 是怎么工作的
#### 4.1 key-range 分片与 Paxos 同步复制
- 数据按 key range 切分
- 每个 split 在多个副本间同步复制
- 写提交天然更贵，但一致性更强
- **主要来源**：
  - `sources/primary/spanner-osdi-2012.pdf`
  - `sources/primary/spanner-life-of-reads-and-writes.md`

#### 4.2 TrueTime：不是“精确时间”，而是“可用的不确定时间”
- TrueTime 暴露的是时间区间，不是假装绝对精确
- commit wait 如何帮助时间戳与真实先后顺序对齐
- external consistency 到底比 serializability 强在哪里
- **主要来源**：
  - `sources/primary/spanner-osdi-2012.pdf`
  - `sources/primary/spanner-true-time-external-consistency.md`
  - `sources/primary/spanner-truetime-cap-theorem.md`

#### 4.3 MVCC、只读事务与快照读
- 为什么只读事务能不阻塞写入
- stale reads 为什么是性能与一致性之间的显式折中
- **主要来源**：
  - `sources/primary/spanner-true-time-external-consistency.md`
  - `sources/primary/spanner-life-of-reads-and-writes.md`

#### 4.4 Spanner 的核心工程哲学：不让所有请求都支付同样代价
- single-split 写更便宜
- multi-split 事务才引入更重的协调与 2PC
- 强读与 stale read 的成本差异
- **主要来源**：
  - `sources/primary/spanner-life-of-reads-and-writes.md`

### 五、Spanner 真的“解决”了吗：它付出了什么代价
#### 5.1 更高的写延迟
- 同步跨机房复制把延迟直接放进提交路径
- 这不是 bug，而是系统选择
- **主要来源**：
  - `sources/primary/f1-vldb-2013.pdf`
  - `sources/primary/spanner-life-of-reads-and-writes.md`

#### 5.2 数据建模变成一等公民
- schema 不再只是逻辑建模，而直接决定物理代价
- locality、主键设计、热点控制的重要性
- 为什么“随便 join”在这种系统里不再廉价
- **主要来源**：
  - `sources/primary/f1-vldb-2013.pdf`
  - `sources/primary/spanner-sql-system-sigmod-2017.pdf`

#### 5.3 它没有绕过 CAP
- 真正网络分区下，仍然要优先 consistency
- Spanner 的“高可用体验”依赖 Google 的网络现实
- TrueTime 不是魔法，只是让某些协调更可控
- **主要来源**：
  - `sources/primary/spanner-truetime-cap-theorem.md`
  - `sources/primary/spanner-true-time-external-consistency.md`

#### 5.4 系统内部复杂度上升
- 从底层事务系统继续演化出 SQL 系统
- distributed query execution、query restarts、range extraction 等能力是不得不补的
- **主要来源**：
  - `sources/primary/spanner-sql-system-sigmod-2017.pdf`

### 六、F1 和后续演化说明了什么
- F1 为什么建立在 Spanner 之上
- 真实业务如何适应同步复制的延迟成本
- 为什么分布式基础设施最终还是会走向更完整的 SQL 能力
- **主要来源**：
  - `sources/primary/f1-vldb-2013.pdf`
  - `sources/primary/spanner-sql-system-sigmod-2017.pdf`

### 七、Spanner 留下的四个核心启发
#### 7.1 分布式数据库不是只靠共识算法就够了
- 真正关键的是复制协议、时间语义、MVCC 与数据模型的组合
- **主要来源**：
  - `sources/primary/spanner-osdi-2012.pdf`
  - `sources/primary/spanner-true-time-external-consistency.md`

#### 7.2 强语义值得做，但必须让代价可见
- 默认强语义
- 提供边界清晰的降级路径
- **主要来源**：
  - `sources/primary/spanner-life-of-reads-and-writes.md`

#### 7.3 真正的突破来自基础设施与系统设计共同进步
- 时钟、网络、复制、调度能力共同支撑了 Spanner
- **主要来源**：
  - `sources/primary/spanner-truetime-cap-theorem.md`
  - `sources/primary/spanner-osdi-2012.pdf`

#### 7.4 后来的 NewSQL，本质上都在回答 Spanner 提出的问题
- 外部一致性怎么逼近
- 跨分片事务怎么降本
- SQL 如何继续成为默认接口
- **主要来源**：
  - `sources/primary/spanner-sql-system-sigmod-2017.pdf`
  - `sources/primary/f1-vldb-2013.pdf`

### 八、结尾：Spanner 的历史位置
- 它最重要的贡献不是某个单点技术
- 而是重新定义了“全球数据库应该如何权衡”
- 收束全文：它没有消灭 trade-off，但把 trade-off 组织成了可工程化的问题
- **主要来源**：
  - `sources/primary/spanner-osdi-2012.pdf`
  - `sources/primary/spanner-truetime-cap-theorem.md`
  - `sources/primary/spanner-sql-system-sigmod-2017.pdf`

## 可选附录
### 附录 A：关键概念小词典
- external consistency
- serializability
- linearizability
- MVCC
- commit wait
- stale reads

### 附录 B：一张图讲清 Spanner 的设计
- Bigtable → Megastore → Spanner → F1
- 或者：分片 / Paxos / TrueTime / MVCC / 事务路径 总览图

## 下一步写作建议
- 先写第二、三、四、五部分，构成文章主体
- 再补第一部分开场和第八部分结尾
- 写作时尽量避免把文章写成“论文摘要串烧”，而要始终围绕一个主线：
  **Spanner 为什么出现、如何成立、代价是什么、为什么它仍然重要**
