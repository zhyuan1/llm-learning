# 文本工件作为可训练参数：Prompt 优化的五年主线

工程团队手里的 prompt、指令、skill 文档、agent 代码，这两年攒成了不小的资产。维护一份还行，几百份就开始失控：改 A 影响
B，没人跑回归，祖传段落谁都不敢动，新人改一行老人盯一周。

能不能像训练模型一样，用「评估 → 改进 → 测试 → 保留或回滚」的循环来持续优化这堆资产？

这是过去五年一条不太显眼的研究主线一直在回答的问题。它分散在 prompt 工程、agent
框架、强化学习、自动机器学习几个圈子里，外人看是十几个不相关的项目，但摆到一张表上会发现它们做的是同一件事。

2022 年 11 月，Yongchao Zhou 等人发了一篇叫 APE 的论文，做了一件听起来不太严肃的事：让 GPT-3 自己写 prompt 给 GPT-3 用。他们在
24 个 NLP 任务上测，结果有 19 个任务，机器写的 prompt 和人类专家写的一样好，甚至更好。其中一个被发现的 prompt 是「Let's think
step by step」，这句后来被无数论文引用。

四年后的 2026 年 5 月，微软研究院发了 SkillOpt 论文，说出了那句把所有线索串起来的话：「把 skill 文档当作 frozen 模型的可训练状态。」

这中间四年，DSPy、TextGrad、OPRO、Promptbreeder、Voyager、Reflexion、ADAS、GEPA、PromptWizard、SkillLens 一个接一个出现。表面看是十几个独立项目，分散在
prompt 工程、agent 框架、强化学习、自动机器学习几个圈子里。但如果把它们摆到同一张表上，会发现一件事：所有这些工作都在做同一回事。

**模型权重冻结，文本工件（prompt、指令、skill 文档、agent 代码）是可训练的参数，验证集得分代替梯度，LLM 改写代替反向传播。**

这是一条贯穿五年的工程主线。它不是某个团队的灵感，也不是某个实验室的偏好，而是当模型本身的更新成本越来越高、参数越来越大、私有
API 越来越多之后，研究者必然会走向的方向。Karpathy 的 autoresearch 在这条线的一端，SkillOpt 在另一端，darwin-skill
是社区里的落地版本。

下面把这条线讲清楚。

---

## 把 LLM 当优化器：APE 起点

APE（Automatic Prompt Engineer）的机制现在看起来朴素到不值得一提：让一个 LLM 提候选 prompt，让另一个 LLM
在开发集上打分，迭代采样表现好的方向。论文里把这个叫「iterative Monte Carlo search」，本质就是黑盒搜索。

但 APE 立下的范式是这条主线的起点。它把三件事放到了同一个循环里：

- 提案（用 LLM 生成候选）
- 评分（用 dev 集得分）
- 保留改进（继续迭代表现好的）

这三件事在所有后续工作里都没变过。变的只是各自的复杂度。

一年后，DeepMind 发了 OPRO（Optimization by PROmpting），把 APE 的思路推到更明确的位置：不是把 LLM 当 prompt 生成器，而是把
LLM 当一个通用的黑盒优化器。给它一段历史「(prompt, score) 对」的轨迹，让它给出下一个候选。OPRO 在 GSM8K 上提了 8 个百分点，在
BBH（Big-Bench Hard）上提了 50%，发现的最优 prompt 之一是「Take a deep breath and work on this problem step by step」。

OPRO 的贡献不在于这个数字，而在于它正式把 prompt 优化建模成「优化器从打分轨迹中学习」。这一步把整个领域从 prompt
工程的玄学里拉了出来，变成了一个有形式化定义的问题。

差不多同时，DeepMind 还做了 Promptbreeder。它做的事情更进一步：不只让 LLM 写任务 prompt，还让 LLM 写「写 prompt 的
prompt」。也就是变异算子（mutation operator）本身也是一个可演化的 prompt。整个系统是一个二层的进化算法，task prompt 和
mutation prompt 一起进化。

到这一步，「LLM 是优化器」这件事被推到极致。优化器不只是一个固定的函数，它本身也是一个可优化的文本工件。

---

## 模块化和文本梯度：DSPy、TextGrad

APE、OPRO、Promptbreeder 都把 prompt 当成一个整体来优化。但实际工程里，一个 LLM 应用通常是多步的：先抽问题、再检索、再推理、再生成答案。每一步都有自己的
prompt，每一步的输出影响下一步的输入。

斯坦福的 Khattab 等人在 2023 年发了 DSPy，把这个问题正式建模。

DSPy 的核心抽象是三个：

- **Signature**：声明输入输出的类型（`question -> answer`）
- **Module**：`Predict`、`ChainOfThought`、`ReAct` 等可组合的执行单元，每个模块的「参数」是它的指令字符串和 few-shot demo 集合
- **Teleprompter / Optimizer**：编译器，给定一个度量函数和一个小训练集，搜索每个模块的最优指令和 demo

`compile` 这一步是 DSPy 最像深度学习的地方。你写出一个由模块组成的 pipeline，调用 `compile`，框架会用 `BootstrapFewShot`、
`COPRO`、`MIPROv2` 等优化器去搜索每个模块的文本参数。最后输出一个固化的、文本参数已经被优化过的程序。

这个过程在概念上和训练神经网络几乎一一对应：你写 forward pass（pipeline 结构），定义 loss（metric），然后调用 `compile`
跑「训练」。区别只是反向传播被换成了对文本的搜索。

DSPy 的实测数字：GPT-3.5 和 Llama2-13B 上的复杂 pipeline，相比标准 few-shot prompting 提升 25% 到 65%。MIPROv2 优化器在
Llama-3-8B 上的多阶段 pipeline 上平均提升约 13%，在 5 个 benchmark 中的 7 个上是当时最优。

但 DSPy 的优化信号还是标量分数。你只知道「这个 prompt 的整体得分是 0.73」，不知道是哪一句话拖了后腿。

TextGrad 把这一步补上了。它在 2024 年由斯坦福的 Yuksekgonul 等人发布，做的事情可以一句话总结：用 LLM 实现反向传播。

每个变量（一个 prompt、一段代码、一个分子描述符）都有它自己的「梯度」。这个梯度不是数值，是自然语言形式的批评：「这一段太长，没有给出具体例子」「这里假设了
user 已经知道概念 X，但你没解释」。这些文本梯度由一个评估 LLM 生成，然后由一个优化 LLM 应用到对应的变量上，写出修改后的版本。

TextGrad 的 API 刻意模仿 PyTorch：`loss.backward()` 把文本梯度沿着计算图传回去，`optimizer.step()` 改写每个变量。

数字上，GPT-4o 在 GPQA 上从 51% 提到 55%，LeetCode-Hard 提升 20%。但更关键的是它把 DSPy 留下的那个空缺补上了：从「整体打分」走到「按变量返回结构化反馈」。

到 TextGrad 这一步，prompt 优化已经有了和神经网络训练几乎对应的全套工具：

| 神经网络  | 文本工件                   |
|-------|------------------------|
| 权重    | prompt / 指令 / few-shot |
| 前向传播  | LLM 调用 pipeline        |
| Loss  | metric / dev 集得分       |
| 反向传播  | LLM 写文本梯度              |
| 优化器步骤 | LLM 改写变量               |
| 验证集   | held-out test set      |

---

## 可训练单元变大：Voyager、ADAS

到目前为止，被优化的都还是「一句指令」或「一组模块的指令」。Voyager 把可训练单元推到了下一个尺度：可执行的代码库。

Voyager 是 NVIDIA 的 Wang 等人 2023 年的工作，背景是 Minecraft。它不是去优化某一个 prompt，而是让一个 GPT-4 驱动的 agent
在游戏里自主探索，把每次成功习得的能力写成 JavaScript 代码，存进一个向量索引的「skill 库」里。后续遇到类似情况，agent 检索最相关的
skill 直接调用。

数字上，Voyager 比之前的 SOTA 多发现 3.3 倍的物品，旅行距离 2.3 倍，科技树进展速度 15.3 倍。但更值得关注的是它确立的范式：*
*agent 在跑的过程中持续累积可执行的 skill，每条 skill 通过自我验证（self-verification）才被加入库**。

Voyager 的验证只是「这次执行有没有报错、有没有达成短期目标」，比后来 SkillOpt 那种 held-out 验证集弱很多。但「skill 库 +
自我验证 + 增量累积」这套框架，到 2025 年 Anthropic 推出 Skills 系统时几乎照搬。

2024 年，Hu 等人的 ADAS（Automated Design of Agentic Systems）把单元再推一层：不优化某个模块的 prompt，不优化某条
skill，而是优化整个 agent 程序。

ADAS 里有一个 meta-agent，它的任务是用 Python 写出 agent 程序，包括 workflow、tool use、prompts 全部。每个候选 agent 在 dev
集上跑一遍，结果存进一个 archive。下一轮 meta-agent 看着 archive 里的历史候选，提出新的 agent。这是「档案驱动的进化搜索」，单元是完整的、图灵完备的代码。

ADAS 发现的一些 agent 设计能跨域迁移、跨基础模型迁移。这说明了一件有意思的事：当可训练单元上升到「完整程序」这个层级，搜索出的解开始有了泛化能力，因为程序结构本身比单条
prompt 更通用。

到这一步，可训练文本工件的尺度已经覆盖：单条指令（APE） → 模块化指令 + demo（DSPy） → 文本梯度更新的任意变量（TextGrad） → 可执行
skill 库（Voyager） → 完整 agent 程序（ADAS）。

---

## GEPA：用反思代替强化学习

2025 年的 GEPA（Genetic-Pareto reflective prompt evolution）把这条主线推到了一个让强化学习社区不太舒服的位置。

它的主张直白：自然语言反思比 RL 微调更高效。

GEPA 的机制：维护一个 prompt 候选群体，每个候选在多个目标上有自己的得分（Pareto 前沿）。每次迭代，选一个候选
prompt，让它实际跑一批任务，把失败的轨迹喂给一个反思 LLM，反思 LLM 读完这些轨迹之后写出改进版本。这是结合了进化算法和
reflection 的一个杂交体。

数字上，GEPA 平均比 MIPROv2 提升 10%，在 AIME-2025 数学题上提升 12%；和 GRPO（一种用 RL 微调模型的方法）比，GEPA 多提 6 到 20
个百分点，但用的 rollout 数量少 35 倍。

35 倍是一个关键数字。它说明：用文本反思更新 prompt，比用 RL 更新模型权重，在样本效率上有数量级差异。

为什么会这样？因为权重更新的信号是标量梯度，每次 rollout 只能传递一个数。而文本反思能把「为什么失败」这件事的结构性信息全部传过去——失败发生在第几步、agent
当时在想什么、本来应该怎么做、应该改 prompt 的哪一句。这种带结构的反馈，每次 rollout 携带的信息量远比一个数值梯度大。

GEPA 是这条主线第一次明确公开宣布：不是在做权重训练的退路，而是在和权重训练正面比效率，并且赢了。

---

## Anthropic Skills：从研究到产品

到了 2025 年 10 月 16 日，Anthropic 正式发布了 Agent Skills 系统。这是这条主线第一次从研究项目变成生产产品。

Anthropic Skills 是一个文件夹结构：每个 skill 由一个 `SKILL.md` 文件 + 可选的脚本 + 可选的资源组成。Claude 在使用 skill
时走「三层渐进披露」（progressive disclosure）：

- 第一层：每个 skill 的 metadata（名字 + 30 到 50 token 的描述），会话开始时全部加载
- 第二层：当某个任务匹配到某条 skill 的描述时，整份 SKILL.md 被加载进上下文
- 第三层：SKILL.md 里链接的脚本、文档、数据，只在子任务真正需要时才被读取

这个三层结构的工程意义是：你可以维护一个很大的 skill 库（成百上千条），但每次只把和当前任务相关的那一份完整加载进上下文，其余只占
30 到 50 token。这让 skill 数量可以无限扩张而不撑爆 context window。

Anthropic Skills 是「运行时」（runtime）。它解决的是：skill 怎么存、怎么发现、怎么按需加载。但它没有解决一个更难的问题：**skill
怎么写才有效？**

这个问题被同年早些时候的 SkillLens 论文摆到了台面上。

---

## SkillLens：诊断为什么 skill 经常没用

微软研究院的 SkillLens（2026 年 5 月，arXiv 2605.23899）做了一件之前没人系统做过的事：测量自动生成的 skill 到底有没有用。

实验在五个领域上跑：ALFWorld（具身智能）、SpreadsheetBench（电子表格）、SWE-bench-Verified（软件工程）、SEAL-0（搜索问答）、BFCL-v4（工具调用）。每个领域里，让一个「提取者」LLM
从历史经验里抽出 skill 文档，再让一个「消费者」LLM 在测试任务上使用这份 skill，看分数变化。

三个发现把整个领域之前的乐观假设打了下来。

**第一，平均有用，但 25% 的情况会变差。** 提取的 skill 在 75% 的实验组合里能提升 agent 表现，但有 25% 出现负迁移（negative
transfer），也就是给 agent 加 skill 之后，它的表现反而比裸跑更差。这个比例足够高，意味着工业界凭感觉推送的 skill
库里，相当一部分实际上在拖后腿。

**第二，LLM 看不出来 skill 的好坏。** 让 GPT-5.4 只看 skill 文本本身，判断质量好坏，准确率 46.4%——和瞎猜没有显著差异。读起来漂亮、结构工整、措辞专业的
skill，跑出来效果有时候反而更差。

**第三，强模型不一定写得出好 skill。** 在 SpreadsheetBench 上，轻量级的 Gemini-3.1-Flash-Lite 在「skill
提取效率」这个指标上排名最高，比基础任务能力更强的 GPT-5.4 还高。会做不等于会教。

SkillLens 同时给出了三个有预测力的维度，是研究者通过元技能（meta-skill）引导下找到的：

- **失败机制编码**（Failure Mechanism Encoding）：把已知的失败路径显式写进去，不只是「别做错」
- **可执行具体性**（Actionable Specificity）：每条指令具体到 agent 不思考就能执行，禁用「视情况而定」「灵活处理」
- **高风险行动黑名单**（High-Risk Action Blacklist）：`rm -rf`、`git reset --hard`、force push 这类操作显式列禁

用这三个维度引导提取，平均提升 1.55 个百分点。用一个看起来合理但没经过验证的通用质量标准引导，效果反而下降 0.59 个百分点。两者相差约
2 个百分点，但方向相反。

这个数字很关键。它说明：凭感觉设计的 skill 评分标准，可能在系统性地把 skill 引导向更差的方向。skill 优化必须基于实证发现的特征，不是基于直觉。

---

## SkillOpt：完整训练循环和验证门控

SkillLens 是诊断，SkillOpt 是治疗方案。

SkillOpt 的核心机制是六步训练循环：rollout → reflect → aggregate → select → update → evaluate。

target agent 跑一批任务（rollout），得到带分数的轨迹。一个独立的 optimizer LLM
分析这些轨迹（reflect），把可能的改动方向汇总（aggregate），从候选里挑最有希望的（select），用它来更新 skill 文档（update），然后用
held-out 验证集打分（evaluate）。

只有当新版 skill 在验证集上的分数严格高于旧版时，更新才被保留。这就是 validation-gated edits（验证门控编辑）。

SkillOpt 还引入了三个让训练稳定的工程机制：

- **文本学习率**：每次更新限制改动的字数和幅度，防止单步改过头
- **拒绝编辑缓冲区**（rejected-edit buffer）：把验证集淘汰的改动记录下来，避免后续轮次反复提同样的无效改动
- **slow / meta update**：每个 epoch 末尾做一次更大幅度的整体重构，避免被局部最优困住

部署的产物是一份 `best_skill.md`，通常 300 到 2000 token。它注入到目标模型的 system prompt 里，不需要改模型权重，不增加推理时的
API 调用。

数字让人有点意外。在 GPT-5.5 上：

- direct chat 场景：平均提升 23.5 个百分点
- Codex CLI（agentic loop）：提升 24.8 个百分点
- Claude Code CLI：提升 19.1 个百分点

6 个 benchmark、7 个目标模型、3 种执行方式，总共 52 个评测格子，SkillOpt 在每一个格子上都是最优或并列最优。

更值得注意的是迁移性：在 GPT-5.5 上训出来的 skill，在更小规模的同系列模型上直接用，效果也比无 skill 好；在 Codex 上训出来的
skill，搬到 Claude Code 上仍然有效；在某个 benchmark 上训出来的 skill，搬到相邻 benchmark 上也保持收益。

这个迁移性是这条主线五年发展的一个收口：训出来的不只是某个模型在某个任务上的最优
prompt，而是一种结构化的领域知识，它在不同模型、不同执行框架、不同任务之间通用。这才像「训练」。

---

## autoresearch 和 darwin-skill 的位置

在这条主线里，Karpathy 的 autoresearch 是用最少的代码把核心机制讲清楚的版本。

autoresearch 的设计：

- `program.md` 是人写的约束和目标（相当于 SkillOpt 里的训练配置）
- `train.py` 是 agent 改的文件（相当于被优化的文本工件）
- `val_bpb`（validation bits per byte）是单一指标
- 5 分钟固定时间预算让不同实验可比较
- git ratchet 只保留让指标下降的修改

autoresearch 的可训练对象是一个 ML 训练脚本，不是 skill 文档。但运行机制和 SkillOpt
完全同构：变量可控、单一度量、validation-gated。一个被优化的是 PyTorch 代码，一个被优化的是 Markdown 文档。机制一模一样。

autoresearch 有个细节值得注意：固定时间预算而不是固定 token 数或 step 数。原因是时间预算跨架构可比较——换了 batch
size、序列长度、attention 模式，5 分钟还是 5 分钟。这个看起来微小的工程选择，背后是「让实验结果可比较」这个隐含目标。这和
SkillOpt 选 held-out 验证集而不是训练集得分作为门控，是同一种工程纪律。

darwin-skill（社区项目，作者花叔）是把这套思路搬到 Claude Code Skill 优化上的落地版本。它和 SkillOpt 的差别在自动化程度上。

SkillOpt 是全自动的，训练循环跑完输出 `best_skill.md`，没有人工确认。darwin-skill 把流程拆成五个阶段，阶段之间强制暂停等用户确认。评分用
9 个维度（包含 SkillLens 的三个新维度），每轮启动两个独立子 agent 评分，下一轮换全新评委（避免锚定效应）。

这个设计差异不是工程偏好，是对「skill 质量能不能被自动评估」这个问题的不同判断。SkillOpt 信任验证集分数，darwin-skill
信任人。哪种合适取决于你的场景里有没有可靠的自动评分。当 skill 的目标是「答对数学题」这种客观可测的任务时，自动门控够用；当
skill 的目标是「写出风格自然的文案」这种主观任务时，人工节点是必要的。

实测上，darwin-skill 把一个图片生成 skill 从 80.8 分优化到 91.65 分，涨幅约 11 分。

---

## 这条主线的几个观察

把这十几个项目摆到一起，有几件事值得想想。

**可训练单元一直在变大。** 从一句指令（APE），到模块化 prompt + demo（DSPy），到任意变量（TextGrad），到完整 skill
库（Voyager、Anthropic Skills），到完整 agent 程序（ADAS、SkillOpt）。粒度上升带来两个变化：一是单次更新能影响 agent
更多行为，二是搜索空间变大、优化变难、需要更强的验证机制。

**优化信号的丰富度也在上升。** 从纯标量得分（APE、OPRO），到带 demo 信息（DSPy），到结构化文本反馈（TextGrad），到带反思的失败轨迹（GEPA）。每一步都在让单次
rollout 携带更多信息，从而需要更少的样本就能收敛。

**验证机制是这条主线最稳定的部分。** 从 STaR（只保留正确推理链）到 autoresearch（git ratchet）到 SkillOpt（validation-gated
edits），名字不一样，做的都是同一件事：不靠看，靠跑。优化器提案，独立的验证集裁决。这条工程纪律比任何具体算法都重要。SkillLens 的
46.4% 自评准确率说得很清楚：你信任 LLM 自己判断「这次改动好不好」，相当于信任随机数生成器。

**LLM 在循环里的角色越来越多。** 早期是「LLM 当被优化的对象」（APE），然后是「LLM 当优化器」（OPRO），然后「LLM
当反向传播」（TextGrad），现在「LLM 当评委」（SkillLens 警告这一步要小心）、「LLM 当 meta-agent」（ADAS）。整个训练流水线已经被 LLM
接管了大半，只有验证步骤的 ground truth（dev 集标签）还是外部的。

**模型权重训练在被这条主线侧面挑战。** GEPA 用 35 倍少的 rollout 就比 GRPO 强 6 到 20 个百分点。SkillOpt 用一份 2000 token
的 Markdown 就能让 GPT-5.5 在 benchmark 上提 19 到 25 个百分点。这两个数字放到一起，在工业落地的语境下意味着：当你想让
agent 表现更好时，先优化 prompt / skill / 文本工件，再考虑微调权重。前者样本效率高一个数量级，且不需要训练基础设施。

---

## 还没解决的问题

这条主线远远没有完成。

**多 skill 之间的相互作用没有研究。** SkillOpt 和 SkillLens 都专注于单一合并型 skill。但 Anthropic Skills 的运行时支持成百上千条
skill 共存，skill 之间可能冲突、重叠、有依赖。一份 skill 的优化会不会干扰另一份，目前没有公开数据。

**主观任务的验证门控没有可靠方案。** 当任务的「分数」需要主观判断时（写作质量、用户体验、设计风格），自动评分本身就是一个没解决的问题。darwin-skill
用「人在回路」绕开这个问题，但这意味着扩展性差。

**优化的 skill 是不是能解释，没人说得清楚。** SkillOpt 的 best_skill.md 是 LLM
写出来的，里面有些条款是有效的（在验证集上提升分数），但人类看不出来为什么有效。这有点像神经网络的可解释性问题，只是发生在文本层面。当你把这种
skill 推到生产，调试一个 bug 时不知道该不该改某条规则，是个真实问题。

**和模型权重训练的边界没划清。** 现在的状况是 prompt 优化和 RLHF / SFT 在很多任务上效果接近，但 prompt
优化不需要训练基础设施。长远看，哪些能力应该放进权重，哪些应该放进文本工件，这条边界目前主要靠工程直觉划，没有理论框架。

但这些都是开放问题，不是路障。这条主线的方向已经很清楚：模型权重在变成「冻结的执行单元」，优化的工作转移到外面那一层。SkillOpt
那句话——「skill as frozen-model trainable state」——不是某个项目的隐喻，是一个五年研究项目的结论。

---

## 参考

- Zhou, Y. et al. [Large Language Models Are Human-Level Prompt Engineers](https://arxiv.org/abs/2211.01910). arXiv:
  2211.01910, 2022. (APE)
- Khattab, O. et
  al. [DSPy: Compiling Declarative Language Model Calls into Self-Improving Pipelines](https://arxiv.org/abs/2310.03714).
  arXiv:2310.03714, 2023.
- Yang, C. et al. [Large Language Models as Optimizers](https://arxiv.org/abs/2309.03409). arXiv:2309.03409, 2023. (
  OPRO)
- Fernando, C. et
  al. [Promptbreeder: Self-Referential Self-Improvement Via Prompt Evolution](https://arxiv.org/abs/2309.16797). arXiv:
  2309.16797, 2023.
- Wang, G. et al. [Voyager: An Open-Ended Embodied Agent with Large Language Models](https://arxiv.org/abs/2305.16291).
  arXiv:2305.16291, 2023.
- Shinn, N. et al. [Reflexion: Language Agents with Verbal Reinforcement Learning](https://arxiv.org/abs/2303.11366).
  arXiv:2303.11366, 2023.
- Yuksekgonul, M. et al. [TextGrad: Automatic "Differentiation" via Text](https://arxiv.org/abs/2406.07496). arXiv:
  2406.07496, 2024.
- Opsahl-Ong, K. et
  al. [Optimizing Instructions and Demonstrations for Multi-Stage Language Model Programs](https://arxiv.org/abs/2406.11695).
  arXiv:2406.11695, 2024. (MIPROv2)
- Hu, S. et al. [Automated Design of Agentic Systems](https://arxiv.org/abs/2408.08435). arXiv:2408.08435, 2024. (ADAS)
- Agrawal, L. A. et
  al. [GEPA: Reflective Prompt Evolution Can Outperform Reinforcement Learning](https://arxiv.org/abs/2507.19457).
  arXiv:2507.19457, 2025.
- Anthropic. [Agent Skills](https://www.claude.com/blog/skills). 2025-10-16.
- Huang, Z. et
  al. [From Raw Experience to Skill Consumption: A Systematic Study of Model-Generated Agent Skills](https://arxiv.org/abs/2605.23899).
  arXiv:2605.23899, 2026. (SkillLens)
- Microsoft Research. [SkillOpt: Executive Strategy for Self-Evolving Agent Skills](https://arxiv.org/abs/2605.23904).
  arXiv:2605.23904, 2026.
- Karpathy, A. [autoresearch](https://github.com/karpathy/autoresearch). GitHub, 2026.
- 花叔 (Alchain). [darwin-skill](https://github.com/alchaincyf/darwin-skill). GitHub, 2026.
