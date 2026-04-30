# Google Spanner Deep Research Digest

## 1. 出现背景：Google 为什么需要 Spanner

Spanner 不是凭空出现的，它是 Google 内部数据系统谱系的延续：

- **Bigtable (OSDI 2006)** 解决的是“超大规模结构化数据存储”问题：面向 PB 级数据、数千台普通服务器、灵活 schema、高吞吐扩展。它非常强，但本质上仍然更偏**可扩展存储系统**，而不是带全局事务语义的数据库。
- **Megastore (CIDR 2011)** 往前迈了一步：在 Bigtable 之上引入了 **Paxos 复制**、跨数据中心副本、以及 **serializable ACID** 语义。但它的事务能力与数据组织方式仍带有明显约束，开发者需要围绕 locality/分组来建模，系统也承担了明显性能代价。
- 到了 Google 广告、配置、元数据等关键业务场景，Google 需要的不再只是“可扩展存储”，而是一个能在**全球范围同步复制**、同时继续提供**像单机数据库一样强的一致性语义**的系统。

一句话概括背景：

> Google 已经解决了“如何把数据存得很大”，但还没有彻底解决“如何把数据在全球范围内复制、事务化，并让应用仍然像写传统数据库那样简单”。

## 2. Spanner 解决了什么问题

Spanner 论文给出的核心目标非常明确：

- 在 **global scale** 上分布数据
- 做 **synchronous replication**
- 仍支持 **externally-consistent distributed transactions**

这背后实际上是在解决四个老问题：

1. **跨地域复制后，怎么还保持强一致事务？**
2. **事务顺序怎么和现实时间顺序对齐？**
3. **读不能因为写而全部阻塞，写也不能因为全局一致性而完全失去可用性能。**
4. **应用不该自己重复处理跨机房复制、冲突顺序、快照一致性这些基础设施问题。**

所以 Spanner 的真正价值，不只是“全球分布式数据库”这几个字，而是：

> 它试图把“全球复制 + 强一致事务 + 工程可用性”同时放进一个系统里。

## 3. Spanner 是如何做到的

### 3.1 数据分片 + 同步复制

Spanner 按 **key range** 切分数据，每个 split 在多个副本之间做 **Paxos 复制**。这意味着它不是先本地提交再异步扩散，而是把复制放到提交路径里。

直接结果：
- 故障恢复和副本切换更干净
- 一致性更强
- 但写路径天然更贵

### 3.2 TrueTime：把“时钟不确定性”显式化

Spanner 最特别的设计不是“用了时间”，而是：

- 它不假装时钟绝对准确
- 它把时间表示成一个带误差边界的区间
- 系统根据这个误差边界做 **commit wait**

这样，Spanner 可以确保：

- 如果事务 T1 在现实世界里先完成，事务 T2 在之后才开始提交
- 那么系统分配给 T2 的时间戳一定大于 T1

这就是 **external consistency** 的关键：

> 不只是结果等价于某个串行顺序，而且这个串行顺序要与外部观察到的提交先后顺序一致。

### 3.3 MVCC + 快照读

Spanner 用 **MVCC** 保存多个版本：

- 写事务创建带时间戳的新版本
- 读事务在某个时间戳上读快照

这样带来两个重要能力：

- **只读事务**不需要像传统锁系统那样大范围阻塞写入
- 可以做 **stale reads / snapshot reads**，在放松新鲜度的条件下换取更高性能

### 3.4 把高代价机制限制在真正需要的时候

官方 whitepaper 很强调一个原则：

> 你不该为没使用的能力付费。

因此：
- **single-split write** 比较便宜，因为不需要完整跨分片 2PC
- **multi-split read-write transaction** 才引入更重的协调、锁和两阶段提交
- **strong reads** 可能需要额外 freshness 协调
- **stale reads** 可以绕开一部分协调成本

这说明 Spanner 不是“无代价的强一致”，而是把成本按场景分层暴露出来。

## 4. Spanner 引入了哪些新问题 / 新代价

### 4.1 更高的写延迟是硬代价

F1 论文说得很直接：

- Spanner 的 **synchronous cross-datacenter replication implies higher commit latency**
- F1 必须通过 **hierarchical schema model** 和 **smart application design** 来吸收这部分成本

也就是说，Spanner 的语义更强，但它不会免费。

### 4.2 数据建模变成性能核心

在传统单机关系库里，schema 设计当然重要，但在 Spanner 这种系统里，**数据是否共址**直接决定事务是否跨 split、是否触发更重的协调。

这带来的新问题是：

- 逻辑上正确的 schema，不一定物理上高效
- 开发者必须更早考虑访问路径、主键布局、局部性和热点
- “随便 join” 不再是廉价操作

换句话说，分布式数据库的性能问题，往往先体现在**主键与数据模型**，而不是 SQL 语句层面。

### 4.3 对基础设施假设更强

Spanner 的 TrueTime 方案背后依赖的是 Google 级别的基础设施条件：

- 较强的时钟同步能力
- 可控的跨数据中心网络
- 很低的长时分区概率

Eric Brewer 在 *Spanner, TrueTime and the CAP Theorem* 里强调的一点很重要：

> Spanner 并没有“绕过 CAP”。

更准确地说：
- 在真实网络分区下，它仍然必须偏向 **consistency over availability**
- 它之所以在生产体验上“看起来很高可用”，是因为 Google 的网络现实让严重分区相对少见

所以 Spanner 的启示不是“CAP 被推翻了”，而是：

> 如果你能极大改善网络与时钟条件，就能把一致性系统的可用性体验做得非常好；但这不是免费午餐。

### 4.4 系统内部复杂度显著上升

后续的 *Spanner: Becoming a SQL System* 说明，Spanner 很快又不得不补上：

- distributed query execution
- query restarts
- range extraction
- 更完整的 SQL 能力
- 更适合混合负载的存储格式

这说明一个现实：

> 当你把更多正确性和分布式复杂度从应用层收回系统层时，系统本身一定会变得更复杂。

## 5. 它带来了哪些启发

### 启发 1：分布式数据库的关键不只是共识，而是“时间 + 数据模型 +副本协议”的组合设计

很多人把 Spanner 的亮点只归结为 Paxos。其实不是。

真正有效的是三者协同：
- Paxos 保证副本提交一致
- TrueTime 让时间顺序可用
- MVCC 把时间戳转成可读快照
- key-range 分片与 locality 设计决定事务成本

### 启发 2：强语义值得做，但要明确告诉用户代价在哪里

Spanner 的一个工程哲学很值得借鉴：

- 默认给用户强语义
- 同时提供有边界的降级路径（例如 stale reads）
- 让更便宜的路径与更昂贵的路径在模型上都清晰可见

这比把系统做成“最终一致但很快”然后把复杂度丢给业务方，更适合关键业务。

### 启发 3：如果基础设施足够强，可以把很多过去认为“不现实”的数据库语义工程化

Spanner 的意义之一，是把原本像论文理想模型的东西，做成了大规模线上系统。

它证明：
- 全球分布不必天然放弃强事务
- 事务系统不必天然放弃扩展性
- 但前提是你愿意为时钟、网络、复制、数据布局付出系统工程成本

### 启发 4：很多“NewSQL”后来都在回答 Spanner 提出的问题

后续很多系统——无论是否真的有 TrueTime——都在试图回答同一组问题：

- 如何在分布式环境中接近外部一致性
- 如何把跨分片事务成本压低
- 如何让 SQL 重新成为分布式系统上的默认接口
- 如何在正确性和延迟之间做显式 trade-off

## 6. 一个简洁结论

如果把 Spanner 放在数据库演化史里看，它最重要的贡献不是某个单点技术，而是这件事：

> 它把“全球复制、强一致事务、数据库可用性”从互相牵制的目标，变成了可以同时追求、但必须精细权衡的系统工程问题。

它没有消灭 trade-off，反而把 trade-off 暴露得更清楚：

- 你可以要更强的一致性
- 你可以要全球复制
- 你也可以保留不错的性能
- 但前提是你接受更高的系统复杂度、更强的基础设施依赖，以及更讲究的数据建模方式

---

## Primary sources used

- `sources/primary/bigtable-osdi-2006.pdf`
- `sources/primary/megastore-cidr-2011.pdf`
- `sources/primary/spanner-osdi-2012.pdf`
- `sources/primary/f1-vldb-2013.pdf`
- `sources/primary/spanner-sql-system-sigmod-2017.pdf`
- `sources/primary/spanner-truetime-cap-theorem.md`
- `sources/primary/spanner-true-time-external-consistency.md`
- `sources/primary/spanner-life-of-reads-and-writes.md`
- `sources/SOURCES.md`
