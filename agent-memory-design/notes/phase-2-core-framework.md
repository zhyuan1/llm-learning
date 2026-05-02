# Agent 记忆设计：Phase 2 核心框架笔记

## 1. 先定一个总判断

Agent 的“记忆”不是单个模块，而是四条并行但彼此耦合的链路：

1. **短期上下文装配**：这一轮给模型看什么
2. **长期记忆写入**：什么信息值得跨会话保存
3. **长期记忆召回**：什么历史信息该在这一轮被拉回来
4. **压缩 / 提炼 / 晋升**：哪些短期痕迹能从流水账变成长期知识

如果只谈“有没有 memory”，很容易把这四件事混成一件事。

---

## 2. 这篇文章最稳的切入线

建议用这条主线：

> Agent 记忆设计，本质上是在处理“上下文窗口有限”与“任务连续性无限”之间的矛盾。

这条线同时被几类来源支撑：

- **MemGPT** 把问题直接表述为有限 context window 与扩展上下文需求之间的矛盾，解法是 tiered memory + virtual context
  management。
- **OpenClaw** 把 context 和 memory 明确拆开：context 是本轮模型输入，memory 是可落盘、可跨轮/跨会话重载的内容。
- **Hermes** 把长期记忆做成 frozen snapshot，把跨会话检索和上下文压缩拆成独立机制。
- **LangMem / CoALA** 提供更稳定的分析语言：要分别回答 What / When / Where，以及 semantic / episodic / procedural 分工。

---

## 3. 先把“上下文”与“记忆”切开

### 3.1 短期上下文是什么

短期上下文不是“我以前知道什么”，而是**当前这一轮实际送进模型窗口的全部材料**。

在 OpenClaw 的定义里，它包括：

- system prompt
- conversation history
- tool calls / tool results / attachments
- workspace 注入文件
- compaction summaries 等压缩产物

这意味着：

- 记忆如果没有被注入这一轮，就不算当前上下文
- 工具 schema、技能列表、系统提示本身也在吃上下文预算
- 讨论 Agent 记忆设计时，不能只看 memory store，还要看 prompt assembly

### 3.2 长期记忆是什么

长期记忆是**被提炼后、可跨轮复用、可跨会话恢复**的信息。

OpenClaw 的表述很直白：模型并不存在隐藏状态，所谓“remember”，本质是写到磁盘并在之后重新装配进来。

Hermes 则更进一步，把长期记忆拆成两个小而硬的常驻块：

- `MEMORY.md`：环境、项目、经验、约定
- `USER.md`：用户画像、偏好、沟通方式

这类设计的关键不是“存很多”，而是“保证高频重要信息始终在场”。

---

## 4. 一个够稳的通用分析框架：What / When / Where / How

LangMem 给了三个核心问题：

- **What**：记什么
- **When**：什么时候形成记忆
- **Where**：存在哪里

我建议再补一个：

- **How**：如何召回、更新、压缩和淘汰

合起来，这四个问题足够解释 OpenClaw、Hermes，也能覆盖 MemGPT 和 CoALA。

### 4.1 What：记什么

按 LangMem / CoALA 的语言，至少可以分三类：

1. **Semantic memory**：事实、偏好、项目约定、稳定知识
2. **Episodic memory**：过去发生过什么、在哪个任务里踩过什么坑
3. **Procedural memory**：应该如何做事、如何回应、什么策略有效

在工程实现里，它们通常长这样：

- **Semantic**：用户偏好、环境事实、项目配置、知识条目
- **Episodic**：会话摘要、任务轨迹、历史案例、日报式笔记
- **Procedural**：system prompt、skills、被反馈修正后的行为规则

### 4.2 When：什么时候写

LangMem 区分两种形成方式：

- **Active / hot path**：在对话过程中立即写
- **Background / subconscious**：对话后或空闲时反思、提炼、写入

这点刚好能拿来解释两个框架的差异：

- **OpenClaw**：同时有 hot path recall，也有 compaction 前 memory flush、dreaming 这类后台晋升机制
- **Hermes**：更强调小而硬的当场写入，再辅以 session search 和外部 memory provider 补容量

### 4.3 Where：存在哪

这里至少有四层：

1. **Prompt 内常驻块**：每轮都直接带上
2. **本地文件 / 结构化 profile**：跨会话持久化
3. **可检索集合**：按需 semantic search / keyword search
4. **完整历史轨迹库**：会话日志、SQLite、长期 transcript store

### 4.4 How：怎么维护

这是很多文章容易写虚的地方，真正的工程差异都在这里：

- 什么时候触发召回
- 召回一条还是召回一组
- 召回靠关键词、向量、混合检索还是显式 profile
- 新信息如何覆盖旧信息
- 何时压缩，压缩到什么粒度
- 哪些短期痕迹能晋升为长期记忆
- 满了之后如何合并 / 替换 / 淘汰

---

## 5. OpenClaw 的设计重点

### 5.1 OpenClaw 不是把 memory 当成一个文件，而是一个分层系统

OpenClaw 至少有三层持久化层次：

- `MEMORY.md`：长期稳定记忆
- `memory/YYYY-MM-DD.md`：日记式短期笔记
- `DREAMS.md`：供人复查的 consolidation / dreaming 输出

这其实已经把“短期痕迹”和“长期知识”分开了。

### 5.2 它最有意思的点，是把 recall 提前到主回复之前

OpenClaw 的 **Active Memory** 不是让主 agent 想起来时再去搜记忆，而是在主回复前先跑一个受限的 blocking memory
sub-agent，把相关记忆塞回本轮上下文。

这背后的判断很重要：

> 大多数 memory 系统的问题不是不会搜，而是搜得太晚。

这使 OpenClaw 特别适合拿来讲“被动记忆”与“主动记忆”的区别。

### 5.3 它的长期记忆晋升链路也很清楚

OpenClaw 的长期记忆写入并不只靠“用户说一句 remember this”：

- compaction 前会先做 **memory flush**，避免上下文压缩前丢失重要信息
- dreaming 会把短期信号做打分和筛选，只把通过阈值的条目晋升进 `MEMORY.md`
- grounded backfill 可以从历史日记里回放并复盘哪些内容值得变成长期知识

所以 OpenClaw 的完整逻辑不是“存储”，而是：

**捕获 → 检索 → 压缩前抢救 → 后台晋升 → 人类复核**

### 5.4 文章里可以怎么写 OpenClaw

如果把 OpenClaw 当例子，它最适合承载这句话：

> OpenClaw 想解决的不是“Agent 怎么存更多”，而是“该出现的记忆能不能在正确的时刻出现”。

---

## 6. Hermes 的设计重点

### 6.1 Hermes 的第一原则是：常驻记忆必须小、硬、稳定

Hermes 的 built-in memory 非常克制：

- `MEMORY.md` 2200 chars
- `USER.md` 1375 chars

它不是无限 memory store，而是一个**强约束、强筛选的常驻前缀层**。

这个设计的价值在于两点：

1. 控制系统提示的固定成本
2. 保持 prefix cache 稳定

### 6.2 Frozen snapshot 是 Hermes 的关键取舍

Hermes 在 session 开始时把记忆注入 system prompt，中途虽然可以写盘，但**不会回写当前 session 的 system prompt**。

这个设计不是缺点，而是性能换取下的有意识取舍：

- prefix 稳定，Anthropic prompt caching 才能持续命中
- 记忆变更不会让 system prompt 每轮抖动
- 代价是新记忆要到下一 session 才进入常驻提示

这点非常适合拿来和 OpenClaw 对照。

### 6.3 Hermes 把“常驻记忆”和“历史召回”明确拆开

Hermes 不是试图用 `MEMORY.md` 解决所有问题，而是把系统拆成两条路：

- **Persistent Memory**：少量关键事实，始终在 prompt 里
- **Session Search**：需要时再去 SQLite/FTS5 历史里搜，再做 LLM summarization

这是很典型的“双层记忆”做法：

- 高频、关键、小容量的信息常驻
- 大容量、低频的信息按需检索

### 6.4 Hermes 对短期上下文管理更“系统软件化”

Hermes 的 context compressor 很值得写，因为它不只是“摘要一下旧对话”，而是完整的上下文预算管理：

- 两层压缩：gateway hygiene 85%，agent compressor 50%
- 先廉价裁剪旧 tool output，再做 LLM summary
- 保留开头固定区和结尾受保护 tail
- 连续压缩时更新已有 summary，而不是每次重写
- 配合 prompt caching 维持稳定的前缀缓存收益

这套设计说明：

> 在 Hermes 里，memory 不是孤立功能，而是和 context compression、prompt caching、session storage 一起设计的。

### 6.5 文章里可以怎么写 Hermes

如果写 Hermes，最值得抓的点不是“它也有长期记忆”，而是：

> Hermes 把长期记忆做成极小而稳定的前缀，把大部分“记得住”的能力交给会话检索、压缩摘要和外部 provider。

---

## 7. OpenClaw 和 Hermes 的差异，不只是功能多少

### 7.1 OpenClaw 更像“记忆编排器”

OpenClaw 的思路更偏：

- 工作区文件天然就是 memory substrate
- recall 可以前置到 reply 之前
- 后台 dreaming 负责从日记和短期信号里做晋升
- memory wiki 又把 durable memory 进一步编译成可追溯知识层

它的关注点是：**让记忆在系统里流动起来**。

### 7.2 Hermes 更像“受控常驻层 + 按需检索层”

Hermes 的思路更偏：

- 常驻层极小
- 中途写入不打扰当前 prompt
- 历史细节交给 session search
- 更深的长期记忆能力通过 provider plugin 外挂
- context compression 和 prompt caching 是一等公民

它的关注点是：**让常驻上下文稳定、便宜、可控**。

### 7.3 一个很适合文章里的对照句

可以把两者差异写成：

- **OpenClaw** 更关心“相关记忆能否及时浮现”
- **Hermes** 更关心“常驻上下文能否长期稳定”

这不是对错之分，是两种优化目标不同。

---

## 8. MemGPT、CoALA、LangMem 能给这篇文章补什么

### 8.1 MemGPT 补的是“为什么 memory 必须分层”

MemGPT 最重要的价值，不是某个具体实现，而是把问题说清楚：

- LLM context window 是稀缺资源
- 但任务、对话、文档是长时程的
- 所以必须像操作系统管理内存一样，做层级化、分页式、虚拟化的上下文管理

这会让文章开头站得更稳。

### 8.2 CoALA 补的是“可以用认知架构语言来解释 Agent”

CoALA 让文章不只停留在工程 tricks，而能把 Agent 记忆放到更通用的认知架构框架里：

- memory modules
- action space
- decision process

用它来讲“记忆不是数据库，而是认知架构的一部分”，很合适。

### 8.3 LangMem 补的是“真正落地时该问哪些问题”

LangMem 最大的价值，是提供一套很适合工程写作的提问方式：

- 记什么
- 何时形成
- 存哪
- 如何检索 / 更新

再加上 semantic / episodic / procedural 的分类，文章就不会只剩下框架对比。

---

## 9. 当前最值得保留的核心判断

下面这些判断，后面进入大纲阶段可以直接复用：

1. **Context 不是 memory。** Context 是本轮模型输入；memory 是可跨轮重用的信息存储层。
2. **Agent memory 至少包含写入、存储、召回、压缩/晋升四个子问题。**
3. **长期记忆不一定越大越好。** Hermes 证明了小而硬的常驻记忆有工程价值。
4. **主动召回比被动检索更像“有记忆”。** OpenClaw 的 active memory 很适合说明这一点。
5. **短期痕迹到长期知识需要中间层。** OpenClaw 的 daily notes / dreaming，Hermes 的 session store / summary，都是在补这层。
6. **真正的差异往往不在存储，而在触发时机与上下文预算管理。**
7. **Agent 记忆设计本质上是一个资源分配问题。** 你在分配 prompt 内稀缺位置、检索时延、写入成本、压缩损失和召回精度。

---

## 10. 暂时不要急着写死的点

这些点还不够稳，Phase 3 写大纲前最好谨慎：

1. **OpenClaw 和 Hermes 哪个“更强”**
    - 现在更像目标函数不同，不适合写成简单优劣比较。

2. **Dreaming / provider plugins 的实际效果评估**
    - 文档说明了机制，但还没有足够一手运行案例支撑“效果很好/不好”这类判断。

3. **MemGPT 与今天这些工程框架的直接继承关系**
    - 可以说它提供了强解释框架，不要轻易写成某项目直接源自它。

---

## 11. 给 Phase 3 的大纲建议

下一阶段大纲可以按这条顺序：

1. 为什么 Agent 需要 memory：上下文窗口有限，但任务是长时程的
2. 先拆概念：context ≠ memory
3. 一个通用框架：What / When / Where / How
4. OpenClaw：主动召回 + 分层晋升
5. Hermes：冻结常驻层 + 按需历史检索
6. 两者对照：不同优化目标下的两种 memory architecture
7. 回到抽象层：MemGPT / CoALA / LangMem 如何帮助我们理解这件事
8. 结尾：Agent 记忆设计，本质上是在设计有限上下文里的注意力制度
