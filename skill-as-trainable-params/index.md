# 像训练神经网络一样训练 Skill

2025 年 10 月 16 日，Anthropic 发布 Agent Skills 系统，把 Claude 用 skill 这件事变成了一个有名字、有规范、有运行时的工程对象。一份
skill 就是一个文件夹，里面放一个 `SKILL.md`，加上一些可选的脚本和资源。Claude 在跑任务时按需加载，不需要全部塞进 system
prompt。

这套系统解决了「skill 怎么用」。它没解决「skill 怎么写才有效」。

七个月后，2026 年 5 月，微软研究院给出了答案，分两篇论文。SkillLens 做的是诊断：自动生成的 skill 在五个领域的实测里有 25% 概率让
agent 表现变差，而且 LLM 自己看 skill 文本判断好坏的准确率只有 46.4%。SkillOpt 做的是治疗：把 skill 文档当作 frozen
模型的可训练状态，用类似深度学习的训练循环优化它，在 GPT-5.5 上把六个 benchmark 平均提升 19 到 25 个百分点。

Karpathy 的 autoresearch 在同一时期出现，是一个最简实现：让 agent 自主修改一个训练脚本，只保留让指标下降的修改。社区里的
darwin-skill 把这套思路搬到 Claude Code Skill 优化上，加了 human-in-the-loop 节点。

把这几件事并排放在一起，会看到一条工程主线：**模型权重冻结，skill 文档（一份 Markdown）是新的可训练参数，验证集得分代替梯度，LLM
改写代替反向传播。**

这篇文章把这条主线讲清楚，每个项目都给一份能跑的安装命令，最后给一份「自己实现 mini 训练循环」的 Python 代码，你可以基于它做扩展。

---

## 一、Anthropic Skills：运行时的三层结构

写 skill 之前，先看 Claude 怎么用 skill。后面讨论「写得好不好」都从这里出发。

Anthropic Skills 的运行时设计叫 progressive disclosure（渐进式披露），分三层。

**第一层：metadata。** 每个 skill 注册时声明一个 name 和一段 30 到 50 token 的 description。Claude 在会话开始时把所有 skill
的 metadata 全部加载进上下文。这一层的总开销是「skill 数量 × ~40 token」。一千条 skill 也只占 4 万 token，对一百万 token 的
context window 来说不成负担。

**第二层：SKILL.md 主体。** 当 Claude 判断当前任务匹配某条 skill 的 description 时，把这条 skill 的 SKILL.md
完整加载进上下文。第二层只对相关的 skill 触发。

**第三层：链接资源。** SKILL.md 里可以链接脚本（`.py`、`.sh`）、文档（`references/foo.md`）、数据文件。这些只在 SKILL.md
主体的步骤明确需要时才被读取。

最小可跑的 SKILL.md 长这样：

```markdown
---
name: invoice-extract
description: 从扫描的发票 PDF 里抽取金额、日期、税号字段，返回 JSON。当用户给一张发票图片或 PDF 并要求提取结构化字段时使用。
---

# Invoice Extract

输入：一张发票（PDF 或图片）
输出：JSON，字段为 amount / date / tax_id / vendor

## 步骤

1. 用 OCR 把图像转成文本（脚本见 references/ocr.py）
2. 按下面的字段映射表抽取关键字段
3. 校验金额格式：必须能转成 Decimal，否则返回错误

## 字段映射

- amount: 找「合计」「总额」「Total」「Amount Due」附近的金额
- date: 找「开票日期」「发票日期」「Date」附近的日期
- tax_id: 找「税号」「Tax ID」「VAT」附近的字符串

## 常见失败

- 印章遮挡金额：返回 amount=null，备注 "occluded by stamp"
- 多页发票：只处理第一页有合计的那一页
```

几个地方注意一下。

**description 字段是召回信号**，Claude 决定要不要加载这条 skill 时，看的就是这一句。写得太抽象（「处理财务文件」）召回不上，写得太具体（「处理北京
2024 年增值税专用发票」）覆盖不全。

**步骤要可执行，不要含糊**。「校验金额格式：必须能转成 Decimal，否则返回错误」是可执行的；「合理校验金额格式」就不是。这一点会在
SkillLens 的实证里被反复印证。

**「常见失败」这一节是 SkillLens 论文里的关键发现**，把已知的失败路径显式编码进去，比单纯写「注意金额可能被遮挡」有效得多。

**装一下试试**：

```bash
# 在 Claude Code 里直接装 Anthropic 官方 skill 集
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/anthropics/skills anthropic-skills

# 看一下官方示例 SKILL.md 怎么写
ls anthropic-skills/skills/
cat anthropic-skills/skills/docx/SKILL.md | head -30
```

或者在 Claude Code 里直接挂载：

```
/plugin marketplace add anthropics/skills
/plugin install document-skills@anthropic-agent-skills
```

到这里，Claude 已经能用上一份 skill。但这份 skill 写得好不好、跑出来效果怎样，是下一节要回答的。

---

## 二、SkillLens：你写的 skill 真的有用吗

微软研究院的 SkillLens（arXiv 2605.23899）是这条主线里第一个系统检验 skill 实际效果的工作。它的实验设计：

- 在五个领域跑：ALFWorld（具身智能）、SpreadsheetBench（电子表格）、SWE-bench-Verified（软件工程）、SEAL-0（搜索问答）、BFCL-v4（工具调用）
- 用「提取者」LLM 从历史经验里抽出 skill 文档
- 让「消费者」LLM 在测试任务上使用这份 skill，对比有 skill 和没 skill 的分数

三个发现。

**发现一，平均有用，但 25% 的情况会变差。** skill 在 75% 的实验组合里能提升 agent 表现，25% 出现负迁移（negative transfer），加
skill 之后 agent 表现反而比裸跑差。也就是说，工业界凭感觉推送的 skill 库里，相当一部分实际上在拖后腿。

这个数字够高，意味着「写一份 skill 大概率有帮助」这个假设站不住。每一份 skill 都需要实测。

**发现二，LLM 看不出来 skill 的好坏。** 让 GPT-5.4 只看 skill 文本本身判断质量，准确率 46.4%，和瞎猜没有显著差异。读起来漂亮、结构工整、措辞专业的
skill，跑出来效果有时候反而更差。

这一条直接否决了一个常见的工程做法：用 LLM 评估 skill。如果你想搭一套「LLM 写 skill + LLM 评分 +
自动迭代」的循环，要知道评分这一步几乎没有信号，整个循环会原地打转。

**发现三，强模型不一定写得出好 skill。** SpreadsheetBench 上，轻量级的 Gemini-3.1-Flash-Lite 在「skill
提取效率」这个指标上排名最高，比基础任务能力更强的 GPT-5.4 还高。会做不等于会教。

什么样的特征对实际效果有预测力？研究者用元技能（meta-skill）引导提取，找到三个稳定有效的维度。

**失败机制编码（Failure Mechanism Encoding）。** 把已知的失败路径显式写进去，不是简单一句「别犯错」。

```markdown
NO: 注意工具可能返回空值
OK: 当 search 工具返回空数组时，agent 历史上会重试三次然后调用 delete_tmp。
这个路径必须截断：第一次返回空数组就停止，向用户报告 "no results"。
```

**可执行具体性（Actionable Specificity）。** 每条指令具体到 agent 不思考就能执行。SkillLens 论文里明确指出，「视情况而定」「灵活处理」「建议尝试」这类措辞在
skill 里是毒药。

```markdown
NO: 根据情况选择合适的解析方式
OK: 文件后缀是 .json 用 json.loads；.yaml 用 yaml.safe_load；
其他后缀返回错误 "unsupported format: {ext}"
```

**高风险行动黑名单（High-Risk Action Blacklist）。** `rm -rf`、`git reset --hard`、`force push` 这类操作显式列禁。

```markdown
## 禁止事项

- 不要用 git reset --hard 回滚，用 git revert
- 不要用 rm -rf 删除目录，先 ls 确认内容再用 rm -i
- 不要 force push 到 main 分支
```

实测数字：用包含这三个维度的 meta-skill 引导 skill 提取，平均效果提升 1.55 个百分点。用一个看起来合理但没经过验证的通用质量评分引导，效果反而下降
0.59 个百分点。

两者相差约 2 个百分点，方向相反。凭感觉设计的评分标准，可能在系统性地把 skill 引导向更差的方向。

到这一步，问题变得很具体：

- 文本看不出好坏，要实测
- 在已经实证的特征上发力（三个维度），不用直觉编标准
- 要有机制把「实测分数」和「skill 改写」连起来

这是 SkillOpt 要解决的问题。

---

## 三、SkillOpt：完整训练循环和验证门控

SkillOpt（arXiv 2605.23904，github.com/microsoft/SkillOpt）的核心是六步训练循环：

```
rollout → reflect → aggregate → select → update → evaluate
```

`target` agent 跑一批任务（rollout），得到带分数的轨迹。一个独立的 `optimizer` LLM
分析这些轨迹（reflect），把可能的改动方向汇总（aggregate），从候选里挑最有希望的（select），用它来更新 skill 文档（update），然后用
held-out 验证集打分（evaluate）。

**只有当新版 skill 在验证集上的分数严格高于旧版时，更新才被保留。** 这是 validation-gated edits（验证门控编辑），整套方案最关键的设计。

为什么这一步关键？不靠门控，优化器会往 skill 里堆越来越多内容。每一条新规则在训练集上几乎总能多救几个
case，所以训练分数一直涨。但这样写出来的 skill 在新数据上会立刻翻车，机器学习领域研究了几十年的过拟合。

加了门控之后，每次更新都要在 held-out 数据上证明有效才能活下来。验证集是裁判，优化器只是提案者。

SkillOpt 还引入了三个让训练稳定的工程机制。

**文本学习率（textual learning rate）**，每次更新限制改动的字数和幅度，比如「最多改两段」「最多增删 200 字」。防止单步改过头，把好不容易稳定下来的
skill 一次性破坏掉。对应神经网络里的 learning rate。

**拒绝编辑缓冲区（rejected-edit buffer）**，把验证集淘汰的改动记录下来，下一轮喂给 optimizer 看：「以下这些方向你已经试过，没用。」这样
optimizer 不会反复提同样的无效改动。对应强化学习里的 negative replay buffer。

**slow / meta update**，每个 epoch 末尾做一次更大幅度的整体重构，避免被局部最优困住。对应学习率衰减或周期性重启。

部署的产物是一份 `best_skill.md`，通常 300 到 2000 token，注入到目标模型的 system prompt 里，不需要改模型权重，不增加推理时的
API 调用。

实测数字（GPT-5.5）：

- direct chat 场景：+23.5 分
- Codex CLI（agentic loop）：+24.8 分
- Claude Code CLI：+19.1 分

在 6 个 benchmark、7 个目标模型、3 种执行方式上跑，总共 52 个评测格子，SkillOpt 在每一个格子上都是最优或并列最优。

迁移性是更关键的一面。GPT-5.5 上训出来的 skill，在更小规模的同系列模型上直接用仍然有效；Codex 上训出来的 skill，搬到 Claude
Code 上仍然有效；某个 benchmark 上训出来的 skill，搬到相邻 benchmark 上也保持收益。

这个迁移性才像「训练」。训出来的不是某个模型在某个任务上的过拟合 prompt，而是结构化的领域知识，跨模型、跨执行框架、跨任务通用。

**装一下试试**：

```bash
# PyPI 安装
pip install skillopt

# 配置 LLM 后端（这里用 Azure OpenAI，OpenAI / Claude / Qwen / MiniMax 都支持）
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
export AZURE_OPENAI_API_KEY="your-key"

# 用仓库自带的 SearchQA split 跑一遍（最小数据集已经在 repo 里）
git clone https://github.com/microsoft/SkillOpt
cd SkillOpt

python scripts/train.py \
    --config configs/searchqa/default.yaml \
    --split_dir data/searchqa_id_split \
    --optimizer_model gpt-5.5 \
    --target_model gpt-5.5 \
    --num_epochs 2
```

跑完会输出 `outputs/<run_name>/best_skill.md`，可以直接拿来用。

config 文件控制了训练循环的所有超参：

```yaml
# configs/searchqa/default.yaml 节选
train:
  train_size: 400
  batch_size: 40
  accumulation: 1
gradient:
  minibatch_size: 8
  merge_batch_size: 8
optimizer:
  learning_rate: 4    # 文本学习率
env:
  name: searchqa
  skill_init: skillopt/envs/searchqa/skills/initial.md
  max_turns: 1
  workers: 24
```

`batch_size`、`learning_rate`、`accumulation`，全是从神经网络训练借过来的。SkillOpt 把 prompt 优化的工程语言对齐到了深度学习。

仓库 ckpt 目录里有训出来的真实 skill。看一份 SearchQA 的 `gpt5.5_skill.md` 节选：

```markdown
# Question Answering Skill

## Concise Answer Normalization

- Prefer the shortest unambiguous answer that directly satisfies the question.
- For natural geographic features, preserve conventional feature designators
  such as "Lake," "River," "Bay," "Gorge," "Mount," or "Island"...

## Common Clue Traps

- Watch for inverse relationships: if the clue says "His third wife was Jiang Qing,"
  the requested answer is the husband, not Jiang Qing.
```

读这份 skill 会发现一件事：里面有相当多条款是「人类不会想到要写」的。「Lake、River、Bay 这些地理类型词要保留」「问 his third wife
时回答的是丈夫不是妻子」，这些都是从大量失败 case 里反推出来的具体规则。靠人工写不出这种规则密度。

「训练」一份 skill 和「写」一份 skill 的差距就在这里。

---

## 四、autoresearch：最简实现

Karpathy 的 autoresearch 不是 skill 优化项目，但它把 SkillOpt 的核心机制用最少的代码讲清楚了。

整个仓库只有三个关键文件：

```
prepare.py    # 固定常量、数据准备、评估函数。不允许修改。
train.py      # 模型、优化器、训练循环。agent 修改这一个文件。
program.md    # agent 的指令文档。人写。
```

`program.md` 长这样（节选自仓库实际内容）：

```markdown
## Setup

1. Agree on a run tag: propose a tag based on today's date
2. Create the branch: git checkout -b autoresearch/<tag>
3. Read prepare.py and train.py for full context
4. Initialize results.tsv with just the header row

## Experimentation

Each experiment runs on a single GPU. The training script runs for
a fixed time budget of 5 minutes (wall clock).

What you CAN do:

- Modify train.py — this is the only file you edit.
  Architecture, optimizer, hyperparameters, batch size — all fair game.

What you CANNOT do:

- Modify prepare.py. It is read-only.
- Install new packages or add dependencies.
- Modify the evaluation harness.

The goal is simple: get the lowest val_bpb.
```

这份 program.md 在概念上对应 SKILL.md，被优化的对象是 train.py（一段 PyTorch 代码），不是 skill 文档。优化机制完全相同：

- 单一指标（val_bpb，validation bits per byte）
- 固定时间预算（5 分钟）让不同实验可比较
- 只允许改一个文件，限制搜索空间
- git ratchet：只保留让指标下降的修改

agent 跑一轮实验，记录到 `results.tsv`：

```
commit    val_bpb     memory_gb   status     description
a3f2b91   0.997900    44.0        keep       baseline
b7c4d22   1.012300    44.1        discard    increased depth to 12
c1e8f45   0.991200    45.3        keep       added muon optimizer for 2D weights
d5a9e11   0.985600    46.0        keep       fused linear+gelu, larger batch
```

每行就是一次完整实验：commit hash、验证集分数、显存、是否保留、改了什么。这就是 SkillOpt 训练循环的最简形式：rollout（跑一次训练）→
evaluate（看 val_bpb）→ keep or discard。

为什么固定 5 分钟而不是固定 step 数？因为时间预算跨架构可比较。换了模型大小、batch size、序列长度、attention 模式，5 分钟还是
5 分钟。如果用 step 数，agent 把模型缩小一半就能跑更多 step，结果就不可比。

这个设计选择背后是一条工程纪律：**让实验结果可比较，比让实验跑得快重要。** 和 SkillOpt 选 held-out 验证集而不是训练集得分作为门控，是同一种思路。

**装一下试试**：

```bash
# 需要一张 NVIDIA GPU（H100 是参考平台，3090/4090 也能跑，参数减小）
# 装 uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# 拉仓库
git clone https://github.com/karpathy/autoresearch
cd autoresearch
uv sync

# 一次性数据准备（约 2 分钟）
uv run prepare.py

# 手动跑一次确认环境通畅（约 5 分钟）
uv run train.py
# 看到最后输出 val_bpb: 0.997xxx 就对了

# 然后启动 Claude Code 或 Codex，让它读 program.md 自主跑
# 一晚上大约 100 轮实验
```

如果你没有 H100，仓库 README 列了几个 fork：`miolini/autoresearch-macos`、`trevin-creator/autoresearch-mlx`（MacOS 版）、
`andyluo7/autoresearch`（AMD 版）。

---

## 五、darwin-skill：human-in-the-loop 的版本

darwin-skill（github.com/alchaincyf/darwin-skill）是把这套思路搬到 Claude Code Skill 优化上的落地版本。它和 SkillOpt
在自动化程度上选了一个不同的位置。

SkillOpt 全自动。训练循环跑完，输出 best_skill.md，没有人工确认。对研究来说没问题，benchmark 上的指标够清晰，自动化合理。

darwin-skill 的判断不一样：skill 的好坏比 validation loss 更微妙，关键阶段要有人参与。整个流程分成五个阶段，阶段之间强制暂停等用户确认。

整体设计：

- **9 维度评估**，包含 SkillLens 论文里的三个新维度（失败机制编码、可执行具体性、高风险行动黑名单）
- **多评委独立审查**：每轮启动两个独立子 agent 评分
- **评委不复用**：下一轮启动全新评委，避免锚定效应
- **早停机制**：单轮涨幅 < 1 分自动停手，避免凑分堆冗余
- **棘轮机制**：新分高于旧分才保留，否则 git revert（禁用 git reset --hard）

实测数据：作者把自己的「图片生成 skill」（huashu-gpt-image）从 80.8 分优化到 91.65 分，涨幅约 11 分。darwin-skill
自身也被自己优化过一轮：86.05 → 92.7。

这个数字比 SkillOpt 的绝对涨幅小，两者场景不同。SkillOpt 测的是标准 benchmark（数学题、问答题），有客观正确答案；darwin-skill
测的是实际使用的 skill，分数本身是 LLM 评委打的，绝对值的可比性有限。

darwin-skill 列了一份反例黑名单，明文禁止 8 个反模式：

1. 同一个 AI 又改又评（呼应 SkillLens 的 46.4% 自评准确率）
2. 用 git reset --hard 当回滚手段（应用 git revert）
3. 为凑分而堆冗余
4. 跳过测试提示词直接评分
5. 一轮内改多个维度
6. 干跑比例 > 30%
7. 静默跳过异常
8. 忽视维度相关簇

这份黑名单本身就是一份「失败机制编码」，把 skill 优化过程中的已知陷阱显式写进去。darwin-skill 用 SkillLens 的方法论优化了自己。

**装一下试试**：

```bash
# 一句话装好（需要 npx）
npx skills add alchaincyf/darwin-skill

# 装好后在 ~/.claude/skills/darwin-skill/ 里
ls ~/.claude/skills/darwin-skill/
# 应该看到 SKILL.md、test-prompts.json、references/ 等

# 然后在 Claude Code 里直接说
# "用 darwin-skill 优化 my-skill，测试集在 ./test-prompts.json"
```

国内访问 GitHub 不稳的情况下可以下载 zip：作者在 README 里给了一个 R2 镜像地址。

---

## 六、把这条主线连起来看

四个项目的位置：

| 层                 | 项目               | 角色                          |
|-------------------|------------------|-----------------------------|
| 运行时               | Anthropic Skills | skill 怎么存、怎么发现、怎么按需加载       |
| 诊断                | SkillLens        | skill 实际效果是什么样、什么特征有效       |
| 训练                | SkillOpt         | 完整训练循环、验证门控、稳定机制            |
| 最简实现              | autoresearch     | 不在 skill 场景，但展示了同一套机制最干净的形式 |
| human-in-the-loop | darwin-skill     | SkillOpt 的工程变体，关键节点暂停等人     |

把这些放到神经网络训练的术语里映射一下：

| 神经网络            | Skill 训练             |
|-----------------|----------------------|
| 模型权重            | SKILL.md 文档          |
| 前向传播            | agent 跑任务（rollout）   |
| Loss            | benchmark 得分或评委评分    |
| 反向传播            | optimizer LLM 分析失败轨迹 |
| 优化器步骤           | LLM 改写 SKILL.md      |
| 学习率             | 文本学习率（限制单次改动幅度）      |
| Negative replay | rejected-edit buffer |
| 验证集             | held-out 任务集         |
| Early stopping  | 单轮涨幅 < 阈值停手          |

每个组件都有对应物。这不是巧合，SkillOpt 论文明确说自己是在「把 frozen 模型权重之外的部分（skill
文档）当作可训练状态」。深度学习社区花了二十年总结出来的工程纪律（验证集裁判、学习率控制、early stopping），在文本工件优化领域被一条一条搬过来。

也有不能直接搬的部分：

- **梯度信号被替换成自然语言反馈**。神经网络的梯度是数值，每次只传递一个数；LLM 的反馈是结构化文本，一次能传递整个失败的归因链。GEPA
  那篇论文（arXiv:2507.19457）实测下来，文本反馈的样本效率比 RL 微调高 35 倍。
- **优化器不是固定的算子，是另一个 LLM**。优化器自己也可以被优化（Promptbreeder 就在做这件事），整个系统比固定算子的优化更灵活。
- **可解释性变差**。SkillOpt 训出来的 skill 里，某些条款是有效的（验证集分数证明了），但人类看不出来为什么有效。有点像神经网络的可解释性问题，只是发生在文本层面。

skill 训练不是神经网络训练的翻版，是借用了它的工程纪律之后的另一套体系。

---

## 七、自己实现一个 mini 训练循环

如果你手头的场景这几个项目都不直接覆盖（比如想优化一个客服回复 skill、一个代码审查 skill、一个数据清洗
skill），可以自己写一份最简的训练循环。

### 方法论：六个工程决定

写代码之前先想清楚六件事，这是把「优化 skill」这个模糊目标转成「能跑的训练循环」需要做的全部决定。

**决定一：把 skill 当唯一可训练参数。** 不动模型，不调采样温度，不改 system prompt 的其他部分。每一轮跑下来，能解释分数变化的只有
SKILL.md 这一份文件。变量越少，归因越准。这对应 SkillOpt 的核心主张，也对应 autoresearch 只让 agent 改 train.py 这一个设计。

**决定二：单一可量化指标。** skill 在测试任务上的平均分。每个任务由 judge LLM 按检查清单（ground_truth）打 0-10
分。指标定下来之后整个训练过程围绕它跑，不要中途切换。指标不准 skill 就训歪，所以指标的设计比训练循环本身重要。

**决定三：训练集和验证集严格分离。** 训练集用来给 optimizer 看失败 case、生成改进方向；验证集只用来打分裁决。两个集合不能有交集，不能互相覆盖。验证集污染是
skill 训练里最隐蔽的失败模式：分数一路涨，部署到真实场景立刻翻车，因为模型其实在背训练集。

**决定四：棘轮（ratchet），只保留改进。** 每次 optimizer 提一个改写候选，在验证集上打分。新分严格大于旧分才 keep，否则
revert。分数只能涨，不能掉。这一步对应 SkillOpt 的 validation-gated edits、autoresearch 的 git ratchet、darwin-skill 的棘轮机制。

**决定五：限制单步改动幅度（textual learning rate）。** 每次只改 1-2 段，不允许整篇重写。一是防止 optimizer 一步把 skill
改坏；二是让分数的归因可追踪——如果新版变差，知道是哪一段引起的。这对应神经网络的 learning rate，幅度太大训练不稳定。

**决定六：用 rejected buffer 给 optimizer 加记忆。** 验证集淘汰的改动方向记下来，下一轮喂给 optimizer 看：「这些方向你已经试过没用。」否则
optimizer 会反复提同样的无效改动，浪费 API 调用。这对应强化学习的 negative replay buffer。

把这六个决定串起来就是一轮训练：

```
读取当前 SKILL.md
  → 在训练集上跑，收集 score < 阈值的失败 case
  → optimizer 看失败 case + rejected buffer，提改写候选
  → 在验证集上给候选打分
  → 新分 > 旧分？keep + 写盘 : revert + 加进 rejected buffer
  → 早停判断（连续 N 轮不涨就停）
  → 下一轮
```

剩下的工作只是把这套逻辑翻译成 Python。

### 实现

按上面六个决定写出来的代码长这样。

### 文件结构

```
my-skill-trainer/
├── skill.md             # 起始 SKILL.md（baseline，自己写一份 70 分的）
├── train_tasks.json     # 训练集任务（用来生成改进方向）
├── val_tasks.json       # 验证集任务（用来打分裁决）
├── trainer.py           # 训练循环
└── outputs/             # best_skill.md、history.json 输出在这里
```

### 训练任务格式

`train_tasks.json` 和 `val_tasks.json` 都是这个格式：

```json
[
  {
    "id": "t01",
    "prompt": "用户问：「我昨天买的耳机怎么还没发货？」",
    "ground_truth": "需要 (1) 道歉 (2) 提供物流查询路径 (3) 给补偿方案"
  },
  {
    "id": "t02",
    "prompt": "用户问：「这个产品支持七天无理由退货吗？」",
    "ground_truth": "需要 (1) 明确说支持 (2) 给出操作入口 (3) 提示注意事项"
  }
]
```

`ground_truth` 不是字面答案，是「评判这次回复对不对」的检查清单。

### 训练循环

`trainer.py`：

```python
import json
import random
from pathlib import Path
from anthropic import Anthropic

client = Anthropic()
MODEL = "claude-sonnet-4-5"

def load_tasks(path):
    return json.loads(Path(path).read_text())

def rollout(skill_text, task):
    """让 target agent 用当前 skill 处理一个任务"""
    msg = client.messages.create(
        model=MODEL,
        max_tokens=1024,
        system=skill_text,
        messages=[{"role": "user", "content": task["prompt"]}]
    )
    return msg.content[0].text

def score(response, task):
    """让 judge LLM 按 ground_truth 打分（0-10）"""
    judge_prompt = f"""判断这次回复是否满足检查清单。
检查清单：{task["ground_truth"]}
回复内容：{response}

返回 0-10 的整数分数。每满足一项 +3 分，不满足扣分。
只返回数字，不要解释。"""
    msg = client.messages.create(
        model=MODEL,
        max_tokens=10,
        messages=[{"role": "user", "content": judge_prompt}]
    )
    try:
        return int(msg.content[0].text.strip())
    except ValueError:
        return 0

def evaluate(skill_text, tasks):
    """在一组任务上跑完，返回平均分"""
    scores = []
    for task in tasks:
        resp = rollout(skill_text, task)
        s = score(resp, task)
        scores.append(s)
    return sum(scores) / len(scores)

def collect_failures(skill_text, train_tasks, threshold=7):
    """收集训练集上得分低的轨迹，作为反向传播信号"""
    failures = []
    for task in train_tasks:
        resp = rollout(skill_text, task)
        s = score(resp, task)
        if s < threshold:
            failures.append({
                "task": task["prompt"],
                "expected": task["ground_truth"],
                "actual": resp,
                "score": s
            })
    return failures

def propose_edit(skill_text, failures, rejected_edits):
    """让 optimizer LLM 提出一个改进版 skill"""
    failure_text = "\n\n".join([
        f"任务: {f['task']}\n期望: {f['expected']}\n"
        f"实际回复: {f['actual']}\n得分: {f['score']}"
        for f in failures[:5]  # 只用最差的 5 个，避免上下文爆炸
    ])
    rejected_text = "\n".join([f"- {r}" for r in rejected_edits[-10:]])

    prompt = f"""你是一个 skill 文档的 optimizer。
当前 skill 文档：
---
{skill_text}
---

下面是 agent 用这份 skill 处理任务时失败的几个 case：
{failure_text}

下面这些方向之前试过，没用，不要再提：
{rejected_text}

请提出一个具体的改进方案：直接给出改写后的完整 skill 文档。
要求：
1. 只改 1-2 段，不要整篇重写（textual learning rate）
2. 必须显式编码失败路径（"当 X 发生时，做 Y，不要 Z"）
3. 禁用模糊措辞（"视情况"、"灵活"、"建议"）
4. 保留 frontmatter

直接输出新的 skill 文档，不要解释。"""

    msg = client.messages.create(
        model=MODEL,
        max_tokens=4096,
        messages=[{"role": "user", "content": prompt}]
    )
    return msg.content[0].text

def train(skill_path, train_path, val_path, num_steps=10, out_dir="outputs"):
    Path(out_dir).mkdir(exist_ok=True)
    skill = Path(skill_path).read_text()
    train_tasks = load_tasks(train_path)
    val_tasks = load_tasks(val_path)
    rejected_edits = []
    history = []

    # 基线分
    best_score = evaluate(skill, val_tasks)
    best_skill = skill
    print(f"[step 0] baseline val_score = {best_score:.2f}")
    history.append({"step": 0, "val_score": best_score, "action": "baseline"})

    for step in range(1, num_steps + 1):
        # rollout + reflect
        failures = collect_failures(best_skill, train_tasks)
        if not failures:
            print(f"[step {step}] no failures on train set, stopping")
            break

        # propose
        candidate = propose_edit(best_skill, failures, rejected_edits)

        # evaluate (validation-gated)
        cand_score = evaluate(candidate, val_tasks)

        # ratchet：只保留改进
        if cand_score > best_score:
            print(f"[step {step}] {best_score:.2f} -> {cand_score:.2f} ✓ keep")
            best_score = cand_score
            best_skill = candidate
            history.append({"step": step, "val_score": cand_score, "action": "keep"})
            Path(f"{out_dir}/best_skill.md").write_text(best_skill)
        else:
            print(f"[step {step}] {best_score:.2f} -> {cand_score:.2f} ✗ revert")
            # 提取改动方向加入 rejected buffer（简化版，实际可以让 LLM 总结）
            rejected_edits.append(candidate[:200])
            history.append({"step": step, "val_score": cand_score, "action": "revert"})

        # early stopping：连续 3 步不涨就停
        recent = history[-3:]
        if len(recent) == 3 and all(h["action"] == "revert" for h in recent):
            print(f"[step {step}] 3 consecutive reverts, stopping")
            break

    Path(f"{out_dir}/history.json").write_text(json.dumps(history, indent=2))
    print(f"\nfinal best_score: {best_score:.2f}")
    return best_skill

if __name__ == "__main__":
    train("skill.md", "train_tasks.json", "val_tasks.json", num_steps=10)
```

### 跑起来

```bash
pip install anthropic
export ANTHROPIC_API_KEY="sk-ant-..."

# 准备 skill.md / train_tasks.json / val_tasks.json
# 训练集 30 个任务，验证集 10 个左右就够起步

python trainer.py
```

输出大概长这样：

```
[step 0] baseline val_score = 5.20
[step 1] 5.20 -> 6.50 ✓ keep
[step 2] 6.50 -> 6.10 ✗ revert
[step 3] 6.50 -> 7.30 ✓ keep
[step 4] 7.30 -> 7.30 ✗ revert
[step 5] 7.30 -> 8.00 ✓ keep
[step 6] 8.00 -> 7.80 ✗ revert
[step 7] 8.00 -> 7.90 ✗ revert
[step 8] 8.00 -> 7.95 ✗ revert
[step 8] 3 consecutive reverts, stopping

final best_score: 8.00
```

`outputs/best_skill.md` 就是训出来的 skill。

### 这份代码做对了什么

把 SkillOpt 的核心机制都包进去了，但删到最简：

- **rollout / reflect / evaluate 三步循环**：`collect_failures` 是 rollout + reflect，`evaluate` 是 evaluate
- **validation-gated edits**：训练集找改进方向，验证集打分裁决，这两个集合严格分离
- **ratchet（棘轮）**：只在 `cand_score > best_score` 时 keep，否则 revert
- **rejected-edit buffer**：失败的方向加入 `rejected_edits`，下一轮告诉 optimizer 别再提
- **textual learning rate**：在 propose_edit 的 prompt 里明确「只改 1-2 段，不要整篇重写」
- **early stopping**：连续 3 步 revert 自动停

每个机制都对应 SkillOpt 论文里的一项。差距在哪：

- **没有 slow / meta update**。SkillOpt 每个 epoch 末尾会做一次大重构跳出局部最优，这版没做。可以加一个分支：每 5 步如果都没涨，让
  optimizer 重写整篇而不是 1-2 段
- **judge 用的是同一个模型**。SkillLens 警告这有自评偏差。生产用的话 judge 换一个不同家的模型（比如 target 用 Claude，judge
  用 GPT-5.5）
- **score 函数粗暴**。让 LLM 返回单个数字，方差大。可以让 judge 返回结构化的逐项检查（每项 0/1），加起来算总分

### 可以扩展的方向

跑通最简版之后，几个值得加的功能：

```python
# 1. 加 batch：每步用 train 集采样 K 个任务，平均分作为信号
def collect_failures_batched(skill_text, train_tasks, batch_size=8, threshold=7):
    batch = random.sample(train_tasks, min(batch_size, len(train_tasks)))
    return collect_failures(skill_text, batch, threshold)

# 2. 加多 judge：避免单 judge 偏差
def score_ensemble(response, task, n_judges=3):
    scores = [score(response, task) for _ in range(n_judges)]
    scores.sort()
    return scores[len(scores) // 2]  # 中位数

# 3. 加 SkillLens 的三个维度静态评分作为辅助信号
def static_score(skill_text):
    """让 judge 评估失败机制编码、可执行具体性、高风险黑名单三个维度"""
    # 完整 prompt 见 darwin-skill 的 SKILL.md
    pass

# 4. 加 git 集成：每次 keep 自动 commit，方便回溯
def commit_skill(message):
    import subprocess
    subprocess.run(["git", "add", "skill.md"])
    subprocess.run(["git", "commit", "-m", f"skill train: {message}"])
```

跑过几轮就会感觉到，整套机制最关键的是验证集，它是裁判，决定哪些改动留下来。验证集准了，optimizer 自己怎么提案影响不大。验证集不准，再聪明的
optimizer 也会走偏。

这一点和神经网络训练完全相同。

---

## 八、几个常见的坑

跑过几次大概会踩这些坑。

**坑 1：测试集太小。** 5 个测试任务跑出来的分数没有统计意义，方差太大。最少 15-20 个任务，覆盖训练集 + 验证集。

**坑 2：训练集和验证集污染。** 如果验证集任务和训练集任务重合，验证分数会虚高。准备数据时严格分离，不要互相覆盖。

**坑 3：评委用的是同一个模型。** SkillLens 实测 LLM 自评准确率 46.4%。如果 skill 的目标模型是 GPT-5.5，评委也用
GPT-5.5，会有自评偏差。最好用一个不同家的模型当评委。

**坑 4：盯着 benchmark 分数，忽略真实任务分布。** 验证集分数升了不一定意味着真实场景表现升了。真实任务往往有验证集覆盖不到的形态。要做线上
A/B。

**坑 5：把 SKILL.md 写成日记。** 优化循环跑久了，SKILL.md 会越来越长。SkillOpt 论文里 best_skill.md 经验值是 300-2000
token。每隔几轮做一次「精简 epoch」，把重叠的规则合并、不再需要的特殊 case 删掉。

**坑 6：把 description 写得太抽象。** description 是 Claude 决定召不召这条 skill
的唯一信号。「处理文本」「分析数据」会被绝大多数任务误召。具体到「从扫描的中文增值税发票里抽取金额、税号、日期」就不会。

---

## 九、还没解决的问题

**多 skill 之间的相互作用。** SkillOpt 和 SkillLens 都专注于单一合并型 skill。但 Anthropic Skills 的运行时支持成百上千条
skill 共存，skill 之间可能冲突、重叠、有依赖关系。一份 skill 的优化会不会干扰另一份？目前没有公开数据。如果你维护一个 skill
库（>50 条），需要自己设计冲突检测机制。

**主观任务的验证门控。** 当任务的「分数」需要主观判断时（写作质量、设计风格、用户体验），自动评分本身就是没解决的问题。darwin-skill
用 human-in-the-loop 绕过这个问题，扩展性差，一份 skill 可以人工验证，一百份就难了。

**优化的 skill 是不是能解释。** SkillOpt 训出来的 best_skill.md 里有些条款明显有效（验证分数证明了），但人类读不出为什么。有点像神经网络的可解释性。生产环境出
bug 时，不知道该改哪一条规则。

**和模型权重训练的边界。** prompt 优化和 RLHF / SFT 在很多任务上效果接近，但成本差一个数量级。GEPA 实测样本效率高 35
倍。哪些能力应该放进权重，哪些应该放进文本工件？目前主要靠工程直觉划，没有理论框架。

---

## 十、把 Skill 当参数训练，意味着什么

回头看这几个项目，最值得记住的不是某个具体技术，而是一个工程心态的转变。

写 skill 之前，**写**这件事被当成创作。坐下来思考用户场景、把经验梳理清楚、用 markdown 排得漂亮、提交到仓库。

把 skill 当参数训练之后，**写**这件事变成了**初始化**。一份 baseline SKILL.md 相当于神经网络的 Xavier
initialization，不需要写得多好，只要不离最优解太远就行。剩下的工作交给训练循环：跑 rollout、收集失败、生成候选、验证、保留改进。一份用得久的
skill 不是某个工程师写出来的，是一套优化循环训出来的。

这个转变在两件事上有现实意义。

**对个人开发者**：不再需要花两天写一份完美的 skill。花两小时写一份 70 分的 baseline，准备 30 个测试任务，用上面那份
trainer.py 跑一晚上，第二天起来就有一份 90 分的 skill。优化的工作量外包给了优化循环。

**对组织**：skill 库可以系统性管理，不只是依赖几个老员工的经验沉淀。SkillLens 的三个维度给了一套可检查的标准，SkillOpt
的训练循环给了一套可扩展的优化流程，darwin-skill 给了一套可介入的人工节点。这些工具让 skill 库的质量管理变得可工程化。

模型权重训练的工程纪律（验证集裁判、学习率控制、early stopping）花了二十年从研究走到生产。skill 训练正在重走同一条路，被优化的对象从张量换成了
markdown，被替换的反向传播换成了 LLM 反馈。

这条路才走了五年。下一个尺度是什么，目前还看不清。但 SkillOpt 那句「skill as frozen-model trainable state」已经把方向钉住了：*
*模型权重在变成冻结的执行单元，优化的工作转移到外面那一层。**

你今天就可以开始跑。

---

## 参考

- Anthropic. [Agent Skills](https://www.claude.com/blog/skills). 2025-10-16.
- Anthropic. [anthropics/skills](https://github.com/anthropics/skills). GitHub.
- Huang, Z. et
  al. [From Raw Experience to Skill Consumption: A Systematic Study of Model-Generated Agent Skills](https://arxiv.org/abs/2605.23899).
  Microsoft Research, 2026. (SkillLens)
- Microsoft Research. [SkillOpt: Executive Strategy for Self-Evolving Agent Skills](https://arxiv.org/abs/2605.23904).
  arXiv:2605.23904, 2026.
- Microsoft Research. [microsoft/SkillOpt](https://github.com/microsoft/SkillOpt). GitHub.
- Karpathy, A. [autoresearch](https://github.com/karpathy/autoresearch). GitHub, 2026.
- 花叔 (Alchain). [darwin-skill](https://github.com/alchaincyf/darwin-skill). GitHub, 2026.
- Agrawal, L. A. et
  al. [GEPA: Reflective Prompt Evolution Can Outperform Reinforcement Learning](https://arxiv.org/abs/2507.19457).
  arXiv:2507.19457, 2025.
