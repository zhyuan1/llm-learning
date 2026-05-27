# Agent Memory 的"做梦"机制——会话间记忆整合

## 文章定位

面向有 Agent 开发经验的技术读者。不做概念科普（"什么是 Agent Memory"），直接切入"为什么 Agent 需要离线整合，怎么做"。

---

## 一、开头：一个直觉——为什么 Agent 跑完一个任务后需要"睡一觉"

Hook: Harvey（法律 AI 公司）给 Agent 开了 Dreaming 后任务完成率提升 6 倍。不是因为模型变强了，是因为 Agent 在两次工作之间"
回顾了一下上次犯过什么错"。

对比：人不是在做事的时候学得最快，是在睡觉的时候——海马体把白天的经历重播、压缩、转存到新皮层。Agent 也是。

提出核心问题：**Agent 的"会话间"发生了什么，才能让它下一次表现更好？**

**来源**: Anthropic blog (Harvey case), Nature Neuroscience 2019

---

## 二、生物学原型：大脑在睡觉时做了什么

不是比喻，是设计参考。三个关键机制：

### 2.1 双系统分工

- 海马体：快速编码当天经历（Agent 的 session）
- 新皮层：慢速整合成长期知识（Agent 的持久化记忆）
- 为什么需要两个系统：单系统学新东西必然覆盖旧东西（catastrophic forgetting 的生物学版本）

### 2.2 睡眠重播（replay）

- Sharp-wave ripples 期间，海马体以 5-20 倍速重播白天经历
- 不是完整回放，是选择性重播（奖励相关的经历优先）
- 重播的目的：让新皮层在不被新输入干扰的条件下慢慢整合

### 2.3 突触缩放（synaptic downscaling）

- 睡眠中全局突触强度下降
- 但相关连接被选择性保留
- 效果：信噪比提高，"重要的变得更突出"

**核心映射**：session = 白天经历；dreaming = 睡眠整合；持久化记忆 = 新皮层存储

**来源**: Nature Neuroscience 2019, Complementary Learning Systems 2024

---

## 三、工程实现：三种"做梦"的技术路线

### 3.1 Anthropic Dreaming（集中式离线审查）

架构定位：

- Managed Agents 独有功能（2026.05 research preview）
- 异步运行于 session 结束后
- 输入：历史 session 的 event log
- 输出：plain-text notes / structured playbooks

工作流程：

1. 回顾过去所有 session 的 transcript
2. 提取跨 session 的 patterns（重复错误、收敛工作流、团队偏好）
3. 整合进记忆（开发者可选自动/手动审核）
4. 后续 session 中自动引用

关键设计决策：

- 建立在 session event log 之上（append-only、不可变、可审计）
- 不修改模型权重——纯粹是记忆层面的操作
- 开发者有控制权（dreaming 发现的 pattern 是否被采纳）

**来源**: Anthropic blog 2026-05-06, Anthropic engineering 2026-04-08

### 3.2 Letta Sleep-time Subagents（分布式自编辑）

架构定位：

- MemGPT 演化而来，开源
- Agent 自己触发、自己执行记忆编辑

工作流程：

1. 触发条件：每 N 条用户消息 / context window 被 compact 时
2. 后台启动 subagent（独立推理链）
3. Subagent 回顾近期对话
4. 直接编辑 MemFS（git-backed markdown 文件系统）
5. 编辑被 commit，可追溯

关键设计决策：

- 记忆是文件系统（不是 vector DB）——Agent 用 bash 工具直接读写
- system/ 目录永远加载进 system prompt；其他文件按需检索
- Subagent 运行 many steps（深度编辑而非简单追加）

与 Anthropic 的区别：

- 去中心化（每个 agent 自己做）vs 集中审查
- 触发是连续的（N 条消息一次）vs 纯 session 间隔
- 粒度更细（文件级编辑）vs 产出 playbook

**来源**: Letta docs 2026-05, MemGPT paper 2023

### 3.3 Mem0 的增量整合（实时无离线）

架构定位：

- 没有显式"离线"阶段
- 每对消息实时抽取、实时整合
- 整合逻辑内嵌在推理链中

工作流程：

1. 每对消息 (m_{t-1}, m_t) 进入抽取函数
2. 结合 conversation summary + 最近 10 条消息作为上下文
3. LLM 提取候选事实 Ω
4. 对每个候选事实，检索 top-10 语义相似的已有记忆
5. LLM 决定四操作之一：ADD / UPDATE / DELETE / NOOP
6. 执行操作

关键设计决策：

- 冲突解决内嵌（不用等离线做）
- Mem0g 标记 invalid 而非物理删除（保留时间线）
- 双检索策略：entity-centric + semantic triplet

与前两者的区别：

- 没有"做梦"阶段——整合是连续的
- 优点：延迟低（p50: 0.148s）、实时一致
- 缺点：无法发现跨 session 的宏观 pattern（只看当前 + 最近 10 条）

**来源**: Mem0 paper (arXiv:2504.19413)

---

## 四、核心设计约束：做梦系统要解决什么问题

### 4.1 什么值得记

- 不是"记住一切"——那和把整个 conversation 塞进上下文没区别
- 筛选标准：出现频率（跨 session 重复）、纠错价值（错过的模式）、泛化性（可迁移到新任务）
- Harvey 案例：法律流程错误是系统性的，发现一个 pattern = 修复全部同类任务

### 4.2 什么时候忘

- 记忆无限增长 → 检索噪音 → 性能退化
- 三种忘的策略：时间衰减、矛盾覆盖、信息量对比（Mem0: 新事实信息量 > 旧记忆时替换）
- 生物学类比：突触缩放——全局降权，选择性保留

### 4.3 一致性如何维护

- 新信息和旧记忆矛盾时怎么办
- Mem0 做法：LLM 判断 → DELETE 旧的 / UPDATE 合并
- Mem0g 做法：标记 invalid 保留历史（"他以前住北京，现在住上海"）
- Anthropic 做法：immutable versioning，所有写入可审计可回滚

### 4.4 谁来做整合

- Agent 自己做（Letta：subagent 模式）→ 去中心化但可能不一致
- 系统做（Anthropic Dreaming）→ 集中但可跨 agent 发现 team-level pattern
- 实时做（Mem0）→ 延迟低但无宏观视野

**来源**: All three framework sources + survey

---

## 五、经济学：什么时候"做梦"比"记住一切"划算

- Claude Opus 4.7 的 1M token 上下文 flat pricing：$5/Mtok input
- 当累计历史 < 500K tokens、session < 10 个时，直接塞上下文更便宜更简单
- 当累计历史 > 10M tokens、session > 100+ 时，显式记忆系统必须存在
- Dreaming 的 ROI 在"任务重复性高 + 跨 session 有可迁移 pattern"的场景最高

数据对比：

- Mem0 vs 全上下文：延迟降 92%，token 成本降 90%+，准确率只低 6%（66.88% vs 72.90%）
- 结论：dreaming 是给"长期运行的生产 Agent"设计的，不是给一次性任务设计的

**来源**: Digital Applied 2026 analysis, Mem0 paper benchmarks

---

## 六、当前局限与未开放的问题

1. **Pattern 质量无法保证**：Dreaming 提取的 pattern 本身可能有错——谁来验证 dreaming 的输出？
2. **跨 agent 整合的隐私问题**：team-level pattern 提取意味着一个 agent 的经验可能被共享——用户是否知情？
3. **记忆中毒（OWASP LLM04/08）**：如果 session 中被注入恶意内容，dreaming 可能将其固化为长期记忆
4. **评估缺失**：目前没有标准 benchmark 评估"dreaming 前后"的差异（Harvey 的 6x 是单一场景）
5. **与持续学习的关系**：Dreaming 只操作记忆，不改权重——这能走多远？parametric memory update 才是终极形态？

**来源**: OWASP LLM Top 10, Survey open challenges, Anatomy of Agentic Memory findings

---

## 七、结尾：Agent 的下一步不是"更聪明"，是"能积累"

回到核心论点：当前 Agent 的瓶颈不在模型智力，在于无法从经验中学习。Dreaming 是第一个不靠重训就让 Agent 持续变好的生产级机制。

它不完美——pattern 可能有错、规模有限、不能跨域泛化。但它代表了一个方向：**Agent 的价值不只来自单次推理的质量，还来自时间的积累
**。

---

## 标题候选

1. `AI Agent 开始"做梦"了——会话间的记忆整合怎么实现`
2. `Agent 不是在工作时变强的——离线记忆整合的三条技术路线`
3. `从海马体到 Dreaming：Agent 如何在"睡觉"时变得更好`
