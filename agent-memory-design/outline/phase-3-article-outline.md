# Agent 记忆设计：文章大纲

## 题目候选

1. **Agent 记忆设计：短期上下文、长期记忆，以及 OpenClaw / Hermes 的两种路线**
2. **Agent 怎么“记住”东西：从 context window 到长期记忆设计**
3. **写给工程师的 Agent Memory：为什么真正的问题不是存储，而是时机与预算**

## 一句话摘要

Agent 记忆设计，不是在系统里加一个 memory store，而是在有限 context window 里，安排哪些信息应该常驻、哪些信息应该按需召回、哪些短期痕迹值得被晋升为长期知识。

## 核心论点

1. `context != memory`，两者是两个层面的问题。
2. Agent memory 至少包含四个子问题：装配、写入、召回、压缩 / 晋升。
3. OpenClaw 和 Hermes 都在做记忆，但优化目标不同。
4. 真正的工程差异，通常不在“存没存”，而在“什么时候出现、出现多少、代价是什么”。

## 目标读者

- 已经在做 AI Agent / coding agent / personal agent 的工程师
- 对 memory 感兴趣，但不想只看概念图的人
- 想从 OpenClaw、Hermes 这些具体系统里抽出通用方法的人

---

## 文章结构

### 0. 开场：为什么 Agent 一定会遇到 memory 问题

**这一节要回答的问题**

- 为什么今天的 Agent 一定绕不过记忆设计
- 为什么“更长的上下文窗口”不能自动解决问题

**核心观点**

- LLM 的 context window 是稀缺资源，但任务、对话、项目协作是长时程的。
- 所以 memory 设计的目标，不是“存更多”，而是“在正确时刻把正确信息放进当前窗口”。
- 这也是 MemGPT 那条线最重要的启发：问题本质上是有限窗口与无限连续性的矛盾。

**可用来源**

- `sources/primary/papers/memgpt-towards-llms-as-operating-systems.pdf`
- `sources/primary/openclaw/openclaw-context.md`
- `sources/primary/hermes/hermes-context-compression-and-caching.md`

**建议写法**

- 用一个很具体的场景开头：用户昨天说过偏好，今天 agent 忘了；或者 agent 明明做过这个项目，但下一轮像第一次见。
- 然后再抛出问题：这到底是“没记住”，还是“没被放进这轮上下文”？

---

### 1. 先拆概念：Context 和 Memory 不是一回事

**这一节要回答的问题**

- 什么是 context
- 什么是 memory
- 为什么很多文章把这两个概念混在一起

**核心观点**

- Context 是这一轮真实送进模型窗口的全部材料。
- Memory 是被保存、可跨轮复用、可跨会话恢复的信息。
- 记忆只有在被装配进当前 prompt 时，才会对当前回答产生作用。

**可用来源**

- `sources/primary/openclaw/openclaw-context.md`
- `sources/primary/openclaw/openclaw-memory.md`
- `sources/primary/hermes/hermes-memory.md`

**建议小节**

- 1.1 OpenClaw 怎么定义 context
- 1.2 OpenClaw / Hermes 怎么定义 persistent memory
- 1.3 一个简单判断：在 disk 上，不等于在本轮 prompt 里

---

### 2. 一个够通用的分析框架：What / When / Where / How

**这一节要回答的问题**

- 工程上该怎么分析一个 Agent 的记忆系统
- 为什么只问“有没有 long-term memory”是不够的

**核心观点**

- `What`：记什么
- `When`：什么时候形成记忆
- `Where`：存在哪里
- `How`：如何召回、更新、压缩、淘汰
- 这四个问题比“短期 / 长期”二分法更适合解释真实工程系统。

**可用来源**

- `sources/primary/frameworks/langmem-conceptual-guide.md`
- `sources/primary/frameworks/langmem-manage-user-profile.md`
- `sources/primary/papers/coala-cognitive-architectures-for-language-agents.pdf`

**建议小节**

- 2.1 What：semantic / episodic / procedural
- 2.2 When：hot path vs background
- 2.3 Where：prompt 常驻层、profile、检索集合、会话历史
- 2.4 How：召回、覆盖、压缩、晋升、淘汰

---

### 3. OpenClaw：把“记忆是否及时浮现”放在第一位

**这一节要回答的问题**

- OpenClaw 的记忆分层是怎么做的
- 它和传统“存下来，等需要时再搜”有什么不同

**核心观点**

- OpenClaw 不是单一 memory 文件，而是分层：`MEMORY.md`、daily notes、`DREAMS.md`。
- 它的重要设计不是单纯存储，而是 recall timing。
- Active Memory 把召回前置到主回复之前，解决的是“搜得太晚”的问题。
- compaction 前 memory flush 和 dreaming 则负责把短期痕迹转成长期知识。

**可用来源**

- `sources/primary/openclaw/openclaw-memory.md`
- `sources/primary/openclaw/openclaw-active-memory.md`
- `sources/primary/openclaw/openclaw-context.md`

**建议小节**

- 3.1 三层持久化：长期、日记、dreaming 输出
- 3.2 Active Memory：为什么主动召回更像“有记忆”
- 3.3 Memory flush：为什么要在 compaction 之前抢救信息
- 3.4 Dreaming / backfill：长期记忆不是一次写入，而是筛选和晋升

**这一节最适合落的一句**
> OpenClaw 关心的不是“存更多”，而是“相关记忆能不能及时浮现”。

---

### 4. Hermes：把“常驻上下文稳定”放在第一位

**这一节要回答的问题**

- Hermes 为什么把 built-in memory 做得这么小
- 它是怎么把长期记忆、历史召回、上下文压缩拆开的

**核心观点**

- Hermes 的 built-in memory 是一个极小但高信号的常驻层。
- `MEMORY.md` + `USER.md` 不是为了包打天下，而是为了让最关键的信息始终在 system prompt 里。
- Frozen snapshot 保住了 prefix cache，也让 system prompt 不会在 session 中途抖动。
- 更深、更广的历史记忆交给 session search 和 external memory providers。

**可用来源**

- `sources/primary/hermes/hermes-memory.md`
- `sources/primary/hermes/hermes-context-compression-and-caching.md`
- `sources/primary/hermes/hermes-readme.md`

**建议小节**

- 4.1 为什么常驻记忆必须小而硬
- 4.2 Frozen snapshot：性能与实时性的取舍
- 4.3 session search：把“常驻事实”和“历史细节”分层
- 4.4 external providers：built-in memory 为什么不是唯一记忆层

**这一节最适合落的一句**
> Hermes 把长期记忆做成稳定前缀，把更多“记得住”的能力交给检索、压缩和外挂 provider。

---

### 5. 别把 memory 单独看：Hermes 的 context compression 其实也是记忆设计的一部分

**这一节要回答的问题**

- 为什么上下文压缩不能被看成 memory 之外的事情
- context compression 到底在 memory system 里扮演什么角色

**核心观点**

- 对 Agent 来说，压缩不是单纯省 token，而是在维护“可继续工作的短期记忆”。
- Hermes 的双层压缩、tail protection、structured summary，本质上都在延长任务连续性。
- Prompt caching 又把“稳定前缀”变成了明确的系统设计约束。

**可用来源**

- `sources/primary/hermes/hermes-context-compression-and-caching.md`
- `sources/primary/openclaw/openclaw-context.md`
- `sources/primary/papers/memgpt-towards-llms-as-operating-systems.pdf`

**建议小节**

- 5.1 压缩不是附属功能，而是短期记忆管理
- 5.2 为什么要保头、保尾、裁中间
- 5.3 为什么 prompt caching 会反过来塑造 memory architecture

---

### 6. OpenClaw vs Hermes：不是谁更强，而是谁在优化不同目标

**这一节要回答的问题**

- 两个系统的差异到底在哪
- 为什么不应该简单做功能表对比

**核心观点**

- OpenClaw 更像“记忆编排器”：强调 recall timing、后台晋升、知识层编译。
- Hermes 更像“受控常驻层 + 按需检索层”：强调稳定前缀、压缩、缓存收益和历史检索。
- 两者都在解决 Agent 记忆问题，只是优化目标不同。

**可用来源**

- `sources/primary/openclaw/openclaw-memory.md`
- `sources/primary/openclaw/openclaw-active-memory.md`
- `sources/primary/hermes/hermes-memory.md`
- `sources/primary/hermes/hermes-context-compression-and-caching.md`

**建议对照维度**

- 常驻记忆层大小
- 主动召回是否前置
- 短期痕迹如何晋升
- 历史信息如何检索
- 和上下文压缩 / prompt caching 的耦合方式

**这一节最适合落的两句**

- OpenClaw 更关心“相关记忆能否及时浮现”。
- Hermes 更关心“常驻上下文能否长期稳定”。

---

### 7. 回到抽象层：MemGPT、CoALA、LangMem 给了我们什么解释框架

**这一节要回答的问题**

- 为什么这些抽象框架值得放进文章
- 它们分别补上了哪一块认知空白

**核心观点**

- MemGPT 让“memory 分层”这件事有了系统软件式的解释。
- CoALA 让 Agent 记忆不只是数据库问题，而是认知架构问题。
- LangMem 把工程实现里真正要问的问题说清楚了。

**可用来源**

- `sources/primary/papers/memgpt-towards-llms-as-operating-systems.pdf`
- `sources/primary/papers/coala-cognitive-architectures-for-language-agents.pdf`
- `sources/primary/frameworks/langmem-conceptual-guide.md`
- `sources/primary/frameworks/langmem-manage-user-profile.md`

**建议小节**

- 7.1 MemGPT：tiered memory / virtual context management
- 7.2 CoALA：memory modules / action space / decision process
- 7.3 LangMem：What / When / Where，以及 profile vs collection

---

### 8. 结尾：Agent 记忆设计，本质上是在设计有限上下文里的注意力制度

**这一节要回答的问题**

- 读完前文后，应该怎么重新理解 Agent memory
- 对工程实践最有用的 takeaway 是什么

**核心观点**

- 真正难的不是把信息存起来，而是决定什么该常驻、什么该按需召回、什么该被遗忘。
- 所谓 Agent 有记忆，不等于它有一个数据库，而是它能在正确时刻，把对当前任务最有帮助的历史带回来。
- 下一代 Agent memory 的关键，不只是更大存储，而是更好的触发、编排和压缩策略。

**可用来源**

- 全文综合

---

## 写作提醒

### 适合保留的判断

- `context != memory`
- memory 至少包含装配、写入、召回、压缩 / 晋升四件事
- 主动召回比被动检索更像“记得住”
- 长期记忆不一定越大越好，小而硬也可能更有工程价值

### 暂时别写死的判断

- 不要直接下结论说 OpenClaw 或 Hermes 谁更强
- 不要把 dreaming / provider plugins 的效果写成已被充分验证
- 不要轻易写成“今天这些系统都直接继承自 MemGPT”

## 下一步

Phase 4 可以按这个顺序填正文：

1. 开场
2. Context vs Memory
3. What / When / Where / How
4. OpenClaw
5. Hermes
6. 对照
7. 抽象框架
8. 结尾
