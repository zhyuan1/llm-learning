# 像训练神经网络一样训练 Prompt

2022 年 11 月，斯坦福的 Yongchao Zhou 等人发了一篇叫 APE 的论文，做了一件听起来不太严肃的事：让 GPT-3 自己写 prompt 给
GPT-3 用。他们在 24 个 NLP 任务上测，21 个任务上机器写的 prompt 和人类专家写的一样好，甚至更好。其中一条被发现的 prompt
是「Let's work this out in a step by step way to be sure we have the right answer.」，把 InstructGPT 在 MultiArith 上从
17.7% 抬到 78.7%。

三年后，2025 年的 GEPA 论文给出一组让强化学习社区不太舒服的数字：用纯文本反思的方式优化 prompt，比 GRPO（一种 RL 微调方法）平均高
6 个百分点、最高 20 个百分点，rollout 用量少 35 倍。Databricks 拿它给企业 agent 调 prompt，开源模型加 GEPA 比 Claude Opus
4.1 便宜 90 倍。

这中间三年，OPRO、Promptbreeder、DSPy、TextGrad、PromptWizard、MIPROv2 一个接一个出现。表面看是七个独立项目，散在 prompt
工程、AutoML、强化学习几个圈子里。把它们摆到同一张表上会发现一件事：**所有这些工作都在做同一回事——模型权重冻结，prompt
是新的可训练参数，验证集得分代替梯度，LLM 改写代替反向传播。**

这是和 Skill 优化同源的一条工程主线。区别在于被训练的对象更小：一段几百 token 的 prompt 字符串，而不是一份几千 token 的
SKILL.md。但机制完全相同：提案、验证、棘轮、文本反馈。

七个项目按时间排列，恰好是「反馈通道带宽逐步放大」的演化史。最早的 APE 通道里只有标量分数，最近的 GEPA
通道里塞的是完整的失败轨迹和反思笔记。每多一倍带宽，需要的 rollout 数就少一个数量级。

这篇文章把这条主线讲清楚，每个项目都给一份能跑的安装命令，最后给一份「自己实现 mini prompt 优化器」的 Python 代码。

---

## 一、APE：把 prompt 优化建模成程序合成

APE（Automatic Prompt Engineer，arXiv:2211.01910）的机制现在看起来朴素到不值一提：让一个 LLM 看几对 `(input, output)`
演示，逆向猜「这条指令是什么」；然后把候选指令拿到 dev 集上打分，分高的留下，分低的淘汰，再围绕高分候选做局部 resample。

但 APE 立下了这条主线的范式。它把三件事放进同一个循环：

- 提案：用 LLM 生成候选 prompt
- 评分：用 dev 集得分
- 保留改进：UCB（Upper Confidence Bound）选择，分配预算给可疑的好候选

这三件事在所有后续工作里都没变过。变的是各自的复杂度。

UCB 的引入是 APE 最值得记的设计。穷举打分太贵，每个候选都跑完整 dev 集需要消耗大量 API 调用。UCB 是一种 bandit
算法，它会自适应地把更多预算分给「目前看起来好但还没充分验证」的候选，把少量预算花在「明显差」的候选上验证它的差。这是把 prompt
搜索建模成 multi-armed bandit 问题的第一次。

**装一下试试**：

```bash
git clone https://github.com/keirp/automatic_prompt_engineer
cd automatic_prompt_engineer
pip install -e .
export OPENAI_API_KEY=...
```

最小可跑的入口是 `simple_ape`：

```python
from automatic_prompt_engineer import ape

# 给几对反义词演示，让 APE 反推「这条指令是什么」
words = ["sane", "direct", "informally", "unpopular", "subtractive"]
antonyms = ["insane", "indirect", "formally", "popular", "additive"]

result, demo_fn = ape.simple_ape(
    dataset=(words, antonyms),
    eval_template='Instruction: [PROMPT]\nInput: [INPUT]\nOutput: [OUTPUT]',
    eval_model='gpt-3.5-turbo-instruct',
    prompt_gen_model='gpt-3.5-turbo-instruct',
    num_prompts=50,        # 候选池大小
    eval_rounds=20,        # UCB 轮数
)
print(result)              # 排序后的候选指令 + 分数
```

跑下来会得到几十条候选指令，APE 自动给出一个排名。最高分那条往往超出人手写的水平。

APE 的局限也很明确。它是纯黑盒，每轮只能看到分数，看不到「为什么错」。如果某条 prompt 在 dev 集上得分 0.3，APE 不知道是因为
prompt 太抽象、还是漏了某类边界 case、还是格式不对。proposer LLM 在下一轮提案时基本是瞎猜，只能靠采样多样性来覆盖搜索空间。

后续所有工作都在补这个洞：把分数轨迹喂回去（OPRO）、把失败 trace 喂回去（GEPA）、把每个变量的局部诊断喂回去（TextGrad）。

---

## 二、OPRO：把分数历史变成 in-context 训练数据

OPRO（Optimization by PROmpting，DeepMind，arXiv:2309.03409）做的事情可以一句话总结：把历史 `(prompt, score)` 对按分数升序写进
meta-prompt，让另一个 LLM 读完之后写出下一条更高分的 prompt。

听起来像 APE 的小修小补，实际上是一次质的变化。APE 的 proposer 看不到分数轨迹，OPRO 的 optimizer 能看到。这相当于把 LLM
当成一个隐式的 regression 模型——读完几十条带分数的样本，外推上升方向。

OPRO 的 meta-prompt 是这样的：

```
Your task is to generate the instruction <INS>. Below are some previous
instructions with their scores. The score ranges from 0 to 100.

text:
Let's solve the problem.
score:
61

text:
Let's think step by step.
score:
71

text:
Take a deep breath and work on this problem step-by-step.
score:
80

Below are some problems. Write your new text that is different from the
old ones and has a score as high as possible.

Problem:
<INS>
Q: {question}
A:
```

每一轮：optimizer LLM 读这段 meta-prompt，写出新的候选指令，在训练集上打分，把 `(新指令, 新分数)` append 回 meta-prompt
末尾。跑两百轮左右饱和。

最有名的发现：在 GSM8K 数学题上，OPRO 找到了 *「Take a deep breath and work on this problem step-by-step.」*，让 PaLM 2-L 达到
80.2%，比 *「Let's think step by step.」* 的 71.8% 高 8.4 个百分点。BBH（Big-Bench Hard）的 23 个子任务里有 19 个超过 baseline
5+ 个百分点。

「深呼吸然后一步一步做」这句话听起来像玄学，但分数轨迹拿到了。这种「人写不出来但确实有效」的指令是 prompt 优化领域的一个经典惊喜。

**装一下试试**：

```bash
git clone https://github.com/google-deepmind/opro
cd opro
pip install -r requirements.txt
```

OPRO 的 repo 里直接有跑 GSM8K 的入口脚本。改一下 API key 和模型名就能跑。

OPRO 的局限：它把 trace 压成 scalar，每条历史记录里只有一个分数。如果一条 prompt 在某些题型上很强、另一些题型上很弱，scalar
平均分会把这种差异抹平。optimizer LLM 看不到「这条 prompt 的强项是数学推理、弱项是组合问题」这种结构化信息。

这个问题要等到 GEPA 用 Pareto frontier + trace 反思才被真正解决。

---

## 三、Promptbreeder：连「怎么变异」也进化

DeepMind 同年（2023）的 Promptbreeder（arXiv:2309.16797）把元层面又抬了一层：不只让 LLM 写 prompt，连「写 prompt 的
prompt」（mutation prompt）也进化。

整个系统是一个二层进化算法。维护一个种群，每个 unit 是 `(task-prompt, mutation-prompt)` 二元组。每代做二元锦标赛选择：两个
unit 比分，输者被胜者用自己的 mutation prompt 改写。但关键创新在第二层：**胜者的 mutation prompt 也会被一个 hyper-mutation
prompt 改写**。

这意味着系统自己在学「怎么改 prompt 最有效」。OPRO 用一个固定的 optimizer 视角看所有问题，Promptbreeder 让 optimizer
视角本身可演化。

PaLM 2-L 上 GSM8K 跑出 83.9%（zero-shot），把 CoT 的 56.4% 和 Plan-and-Solve 的 60.5% 远远甩开。最佳进化 prompt 是一段唠叨的「Show
all your working. II. ... III. ... IV. ...」——没人会这么写，但分数最高。

**装一下试试**（无官方代码，社区实现 `vaughanlove/PromptBreeder`）：

```bash
git clone https://github.com/vaughanlove/PromptBreeder
cd PromptBreeder
pip install -r requirements.txt
export COHERE_API_KEY=...
```

调用 entry point：

```python
from pb import create_population, init_run, run_for_n
from pb.mutation_prompts import mutation_prompts
from pb.thinking_styles import thinking_styles
import cohere, os

co = cohere.Client(api_key=os.environ['COHERE_API_KEY'])

# 2 mutation prompts × 4 thinking styles = 8 个初始 unit
population = create_population(
    tp_set=mutation_prompts[:2],
    mutator_set=thinking_styles[:4],
    problem_description="Solve the math word problem, "
                        "giving your answer as an arabic numeral.",
)

init_run(population, co, num_evals=10)               # 初始评分
run_for_n(n=10, population=population, model=co, num_evals=10)   # 跑 10 代

print(population.units)   # 最终 task-prompt + mutation-prompt + 分数
```

Promptbreeder 的代价是它的最大短板：极其昂贵。PromptWizard 论文里测过 Promptbreeder 在 BBII 上要 18,600 次 API 调用、1.488M
tokens，相当于跑一遍要烧几十美元。后面的 PromptWizard 用反思代替盲改，把成本压到 270 倍以下。

---

## 四、DSPy：把 prompt 优化抽象成「编译程序」

到这里，APE / OPRO / Promptbreeder 都还在优化「一句指令」。但实际工程里，一个 LLM 应用通常是多步的：先抽问题、再检索、再推理、再生成答案。每一步都有自己的
prompt，每一步的输出影响下一步的输入。

斯坦福的 Khattab 等人 2023 年发了 DSPy（arXiv:2310.03714），把这个问题正式建模。

DSPy 的核心抽象是三个：

- **Signature**：声明输入输出的契约（`question -> answer`）
- **Module**：可组合的执行单元（`Predict`、`ChainOfThought`、`ReAct`），每个模块的「参数」是它的指令字符串和 few-shot demo 集合
- **Teleprompter / Optimizer**：编译器，给定 metric 和小训练集，搜索每个模块的最优指令和 demo

`compile()` 这一步是 DSPy 最像深度学习的地方。你写一个由模块组成的 pipeline，调用 `compile`，框架会用 `BootstrapFewShot`、
`MIPROv2`、`COPRO` 等优化器去搜索每个模块的文本参数。最后输出一个固化的、文本参数已经被优化过的程序。

这个过程在概念上和训练神经网络几乎一一对应：你写 forward pass（pipeline 结构），定义 loss（metric），调用 `compile`
跑「训练」。区别只是反向传播被换成了对文本的搜索。

**装一下试试**：

```bash
pip install dspy
export OPENAI_API_KEY=...
```

最小可跑代码：

```python
import dspy

dspy.configure(lm=dspy.LM("openai/gpt-4o-mini"))

# 1. Signature：声明式 I/O 契约
class GenerateAnswer(dspy.Signature):
    """Answer math word problems with a single integer."""
    question: str = dspy.InputField()
    answer: str  = dspy.OutputField(desc="arabic numeral only")

# 2. Module：组合 CoT 推理器
qa = dspy.ChainOfThought(GenerateAnswer)

# 3. Metric：验证集分数 = 梯度替身
def exact_match(example, pred, trace=None):
    return pred.answer.strip() == example.answer.strip()

trainset = [dspy.Example(question=q, answer=a).with_inputs("question")
            for q, a in load_gsm8k_train()]

# 4a. 便宜优化器：bootstrap few-shot demos
boot = dspy.BootstrapFewShot(metric=exact_match, max_bootstrapped_demos=4)
qa_v1 = boot.compile(student=qa, trainset=trainset)

# 4b. 强优化器：MIPROv2 联合搜索 instructions + demos
mipro = dspy.MIPROv2(metric=exact_match, auto="medium")
qa_v2 = mipro.compile(student=qa, trainset=trainset,
                      requires_permission_to_run=False)

print(qa_v2(question="A bat and ball cost $1.10. The bat is $1 more...").answer)
```

`BootstrapFewShot` 是基础优化器：它跑一遍训练集，把 student 模型自己答对的轨迹收集起来当 few-shot
demo。这是一种自举（bootstrap）：用模型自己的成功输出当训练数据，不需要人工标注。

`MIPROv2`（Multi-prompt Instruction PRoposal Optimizer v2，arXiv:2406.11695）是当前主力优化器。它做两件事：用 bootstrap 收集种子
demo，用贝叶斯优化（基于 Tree-structured Parzen Estimator）联合搜索「指令文本 × demo 子集」。这是离散版本的 hyperparameter
optimization，把 prompt 优化做得像 AutoML。

实测数字：MIPROv2 在 7 个多阶段 LM 程序上用 Llama-3-8B 比基线 teleprompter 高最多 13% 准确率。

DSPy 的范式转折在于：它把 prompt 优化从「调字符串」抬到了「编译程序」。Signature × Module × Metric × Compile
这四个抽象一旦形式化，任何下游优化器都可以即插即用。后来的 GEPA 直接作为 dspy 的一个新优化器接口（`dspy.GEPA`）发布，无缝接管原本
MIPROv2 的位置。

DSPy 的局限：MIPROv2 仍然把 trace 压成 scalar metric，optimizer 不会「读」错误。这是下一节 TextGrad 要补的洞。

---

## 五、TextGrad：用 LLM 实现反向传播

斯坦福 Yuksekgonul 等人 2024 年的 TextGrad（arXiv:2406.07496，*Nature* 2025）做了一件让人惊讶的事：把 PyTorch autograd
的语义直接搬到文本域。

每个文本变量是一个 `tg.Variable`。前向是 LLM call。loss 是另一个 LLM 评估器输出的批评文本。`loss.backward()` 让评估器写出*
*针对每个上游变量的 textual gradient**——一段自然语言形式的批评，比如「这段答案错在 X，应该改 Y」。`optimizer.step()` 用一个
TGD（Textual Gradient Descent）prompt 把变量按这些 gradient 改写。

API 刻意模仿 PyTorch：

```python
import textgrad as tg
tg.set_backward_engine("gpt-4o", override=True)

# 1. 前向：让 LLM 给一个初始答案
question = tg.Variable(
    "If 25 shirts dry in 1 hour, how long for 30 shirts? Reason step by step.",
    role_description="question to the LLM",
    requires_grad=False,
)
model  = tg.BlackboxLLM("gpt-4o")
answer = model(question)
answer.set_role_description("concise and accurate answer")

# 2. Loss = 一段自然语言 rubric，由 LLM 评估
loss_fn = tg.TextLoss(
    "Evaluate this answer. Be smart, logical, very critical. "
    "Concise feedback only."
)
loss = loss_fn(answer)

# 3. Backward + step：textual gradient 反向流动，answer 被改写
optimizer = tg.TGD(parameters=[answer])
loss.backward()
optimizer.step()

print(answer.value)   # 改写后的答案（1 hour，不是错误的 1.2）
```

**装一下试试**：

```bash
pip install textgrad
export OPENAI_API_KEY=...
```

实测数字：GPT-4o 在 GPQA（Google-Proof QA）上从 51% 提到 55% 零样本；LeetCode-Hard 提升 20% 相对增益。论文里还展示了同一 API
优化分子结构和放疗治疗计划——可微的不只是 prompt。

TextGrad 是这条主线第一次把「反馈」从标量升级成结构化对象。神经网络的梯度是数值，每次只传递一个数。TextGrad
的梯度是一段诊断文本，一次能传递整个失败的归因链。这种带结构的反馈，每次 rollout 携带的信息量远比一个数值梯度大。

它的局限是逐样本 SGD 风格的更新会震荡，缺少全局多样性管理。如果某个样本特别难，TGD 会反复围绕它震荡，把 prompt 改坏。这是
GEPA 用 Pareto frontier 解决的问题。

---

## 六、PromptWizard：Critique 不打分，定位失败模式

微软 2024 年的 PromptWizard（arXiv:2405.18369，github.com/microsoft/PromptWizard）做的是「精修」：不要群体进化，靠少量精确改写代替。

两阶段。第一阶段优化指令：用思维风格（thinking styles）多样化指令变体，让 LLM 在 mini-batch 错例上**批评**自己刚生成的 prompt
哪里不对，再合成一版。第二阶段优化 in-context examples：用 critique 找到当前 prompt 在哪类样本上薄弱，针对性合成正例、负例、合成例子，再接
CoT 推理链。

整个过程没有大种群，靠 critique-and-synthesize 的局部精修代替进化搜索。

PromptWizard 的关键洞见：**critique 不输出分数，输出失败模式定位**。OPRO 看到「这条 prompt 得 0.3 分」，PromptWizard 看到「这条
prompt 在涉及单位换算的题型上一致失败，因为指令里没强调先把单位统一」。一个是数字反馈，一个是定性反馈。后者直接告诉 optimizer
「补哪个 demo」。

**装一下试试**：

```bash
git clone https://github.com/microsoft/PromptWizard
cd PromptWizard
pip install -e .
# 设 OPENAI_API_KEY 或 AZURE_* 在 .env
```

最小调用：

```python
from promptwizard.glue.promptopt.instantiate import GluePromptOpt
from promptwizard.glue.promptopt.techniques.common_logic \
    import DatasetSpecificProcessing

class GSM8K(DatasetSpecificProcessing):
    def extract_final_answer(self, text):
        return text.split("<ANS_START>")[-1].split("<ANS_END>")[0].strip()

gp = GluePromptOpt(
    promptopt_config_path="demos/gsm8k/configs/promptopt_config.yaml",
    setup_config_path="demos/gsm8k/configs/setup_config.yaml",
    dataset_jsonl="demos/gsm8k/data/train.jsonl",
    data_processor=GSM8K(),
)

best_prompt, expert_profile = gp.get_best_prompt(
    use_examples=True,
    run_without_train_examples=False,
    generate_synthetic_examples=False,
)
print(best_prompt)
```

实测数字：GPT-4 在 GSM8K 上 95.4%；BBII 上仅 69 次 API 调用、24K tokens——相比 Promptbreeder 的 18,600 次调用、1.488M
tokens，成本降低 270 倍，准确率仍领先。

PromptWizard 的局限：单候选精修没有 Pareto 多样性。如果任务内有多个不兼容子分布（比如数学题里既有几何也有代数），精修一边就会拖累另一边，容易
mode collapse。GEPA 的 Pareto frontier 正是补这个洞。

---

## 七、GEPA：Pareto 前沿 + 完整 trace 反思

GEPA（Genetic-Pareto reflective prompt evolution，arXiv:2507.19457，2025）是这条主线目前的终点。

每一轮：

1. 从 **Pareto frontier**（在某个 task 子集上当前最强的候选合集）里选一个 parent
2. 在 minibatch 上跑，**收集完整 trace**：推理日志、错误信息、工具输出
3. 让 reflection LLM **读 trace** 诊断为什么失败、提出针对性修改
4. 接受改进者，更新 Pareto front

可选还有 **system-aware merge**：合并两个分别擅长不同 task 的候选。

GEPA 把前面所有项目的优点组合起来：Promptbreeder 的进化 + PromptWizard 的 critique-then-synthesize + TextGrad
的结构化反馈 + 一个新东西——**Pareto frontier 的多样性管理**。

什么是 Pareto frontier？想象你有 100 道训练题，每个候选 prompt 在每道题上都有一个 0 或 1 的得分。一个候选 A「在 80 道题上对、20
道题上错」，候选 B「在 70 道题上对、30 道题上错，但 B 对的 30 道题 A 错了」。在 OPRO 的 scalar 平均分体系里 A 严格优于 B（80 >
70），B 会被淘汰。在 Pareto frontier 体系里 B 不会被淘汰，因为 B 在「A 错的那 30 道题」上是当前最强的，这种 specialization
有保留价值，可能在 merge 阶段贡献给某个新候选。

这个机制让 GEPA 不会陷入「平均分最高但缺乏多样性」的局部最优。

**装一下试试**：

```bash
pip install gepa
```

最小可跑（来自官方 README）：

```python
import gepa

trainset, valset, _ = gepa.examples.aime.init_dataset()

seed = {
    "system_prompt":
        "You are a helpful assistant. Answer the question. "
        "Put your final answer in the format '### <answer>'"
}

result = gepa.optimize(
    seed_candidate=seed,
    trainset=trainset,
    valset=valset,
    task_lm="openai/gpt-4.1-mini",      # 便宜的执行器
    reflection_lm="openai/gpt-5",        # 强反思器读 trace
    max_metric_calls=150,                # 比 GRPO 便宜 35×
)

print(result.best_candidate["system_prompt"])
```

注意架构：`task_lm` 跑任务，可以用便宜模型；`reflection_lm` 读 trace 做诊断，用强模型。这个分工让 GEPA
在成本上有一档优势——执行器跑成百上千次，反思器只跑几十次。便宜的地方使用便宜模型，贵的地方使用贵模型。

`gepa.optimize_anything` 接受任意文本工件 + 自定义 evaluator——GEPA 不绑定 prompt，连代码、agent 架构、调度策略都能进化。

实测数字：

- 6 个任务上比 MIPROv2 平均高 10%，AIME-2025 上高 12%
- 比 GRPO（一种 RL 微调方法）平均高 6%、最高 20%
- rollout 用量减少最多 35 倍
- GPT-4.1 Mini 在 AIME 2025 用 150 次 metric call 从 46.6% 提到 56.6%
- Databricks 实测：开源模型 + GEPA 在企业 agent 任务上比 Claude Opus 4.1 便宜 90 倍

35 倍是这条主线最有冲击力的数字。它说明用文本反思更新 prompt，比用 RL 更新模型权重，在样本效率上有数量级差异。

为什么会这样？因为权重更新的信号是标量梯度，每次 rollout 只能传递一个数。文本反思能把「为什么失败」这件事的结构性信息全部传过去——失败发生在第几步、agent
当时在想什么、本来应该怎么做、应该改 prompt 的哪一句。这种带结构的反馈，每次 rollout 携带的信息量远比一个数值梯度大。

GEPA 是这条主线第一次明确公开宣布：不是在做权重训练的退路，而是在和权重训练正面比效率，并且赢了。

---

## 八、把这条主线连起来看

七个项目按时间排，正好是「反馈通道带宽」单调放大的过程。

| 项目             | 反馈信号                            | 信息密度              |
|----------------|---------------------------------|-------------------|
| APE            | 单一标量分数 + UCB                    | 每轮 1 bit          |
| OPRO           | 分数历史轨迹                          | 每轮 N 个数           |
| Promptbreeder  | 锦标赛胜负 + 元层 mutation             | 每轮 1 bit + 隐式策略学习 |
| DSPy / MIPROv2 | 标量 metric + bootstrapped demos  | 每轮 1 数 + demo 集合  |
| TextGrad       | 每变量自然语言批评                       | 每轮一段诊断文本          |
| PromptWizard   | Critique 定位失败模式                 | 每轮一份失败模式报告        |
| GEPA           | 完整 trace + 反思 + Pareto frontier | 每轮一份诊断报告 + 多目标向量  |

把这些项目放到神经网络训练的术语里映射一下：

| 神经网络            | Prompt 训练                   |
|-----------------|-----------------------------|
| 模型权重            | prompt / 指令 / few-shot demo |
| 前向传播            | LLM 调用 pipeline             |
| Loss            | metric / dev 集得分            |
| 反向传播            | LLM 写文本梯度                   |
| 优化器步骤           | LLM 改写 prompt               |
| 学习率             | 单步改动幅度限制                    |
| Negative replay | rejected-edit buffer        |
| 验证集             | held-out test set           |
| Early stopping  | 单轮涨幅 < 阈值停手                 |

每个组件都有对应物。这不是巧合——DSPy 的 `compile()` 概念明确借鉴了 PyTorch 的 `nn.Module.compile()`，TextGrad 直接用
`loss.backward()` + `optimizer.step()`。深度学习社区花了二十年总结出来的工程纪律，被一条一条搬到了文本工件优化领域。

但也有不能直接搬的部分。

**反馈信号的丰富度根本不同**。神经网络的梯度是数值，文本工件的「梯度」是诊断报告。后者每次 rollout 携带的信息量高一两个数量级。这是
GEPA 比 GRPO 便宜 35 倍的根源。

**优化器不是固定算子**。Adam、SGD 的更新规则是数学公式，固定不变。Prompt 优化器是另一个
LLM，每次调用都可能给出不同的改写。这意味着优化器自己也可以被优化——Promptbreeder 已经证明这条路。

**可解释性变差**。MIPROv2 选出来的最优 prompt、GEPA 进化出来的最强候选，里面有些条款人类读不出为什么有效。比如「Take a deep
breath and work on this problem step-by-step」，没人能给出理论解释为什么深呼吸能让数学分数涨 8
个百分点。这是文本层面的可解释性问题，类似神经网络的黑盒问题。

---

## 九、自己实现一个 mini prompt 优化器

如果你的场景这几个项目都不直接覆盖（比如优化一段客服开场白、一段 agent 的 reasoning prompt、一段 SQL 生成
prompt），可以自己写一份最简的训练循环。

### 方法论：六个工程决定

写代码之前先想清楚六件事，这是把「优化 prompt」这个模糊目标转成「能跑的训练循环」需要做的全部决定。

**决定一：把 prompt 当唯一可训练参数**。不动模型，不调采样温度，不改其他的输入处理。每一轮跑下来，能解释分数变化的只有 prompt
这一个变量。变量越少，归因越准。

**决定二：单一可量化指标**。每个测试任务都要有一个客观打分。最稳的形式是「检查清单评分」：每个任务带一个 ground_truth
检查清单，judge LLM 看回复满足几条，按比例给分。比让 LLM 直接打 0-10 分稳得多。

**决定三：训练集和验证集严格分离**。训练集用来给 optimizer 看失败 case、生成改进方向；验证集只用来打分裁决。两个集合不能有交集。验证集污染是
prompt 优化里最隐蔽的失败模式：分数一路涨，部署到真实场景立刻翻车。

**决定四：棘轮，只保留改进**。每次 optimizer 提一个改写候选，在验证集上打分。新分严格大于旧分才 keep，否则 revert。分数只能涨，不能掉。

**决定五：用结构化 critique 代替标量打分作为 optimizer 输入**。这是这条主线从 OPRO 走到 GEPA 学到的最重要的事。给 optimizer
的不是「这条 prompt 得 0.3 分」，而是「这条 prompt 在涉及 X 类型的任务上一致失败，因为指令没说清 Y」。

**决定六：rejected buffer 给 optimizer 加记忆**。验证集淘汰的方向记下来，下一轮告诉 optimizer 别再提。否则会反复浪费 API 调用。

把这六个决定串起来就是一轮训练：

```
读取当前 prompt
  → 在训练集上跑，收集失败 case
  → judge LLM 写 critique（不只是打分，要定位失败模式）
  → optimizer 看 critique + rejected buffer，提改写候选
  → 在验证集上给候选打分
  → 新分 > 旧分？keep + 写盘 : revert + 加进 rejected buffer
  → 早停判断
  → 下一轮
```

剩下的工作是把这套逻辑翻译成 Python。

### 实现

按上面六个决定写出来的代码长这样。

```python
import json
from pathlib import Path
from anthropic import Anthropic

client = Anthropic()
MODEL = "claude-sonnet-4-5"

def load_tasks(path):
    return json.loads(Path(path).read_text())

def rollout(prompt, task):
    """让目标模型用当前 prompt 处理一个任务"""
    msg = client.messages.create(
        model=MODEL,
        max_tokens=1024,
        system=prompt,
        messages=[{"role": "user", "content": task["input"]}],
    )
    return msg.content[0].text

def score_with_critique(prompt, task, response):
    """让 judge LLM 同时返回分数和失败模式定位"""
    judge_prompt = f"""判断这次回复是否满足检查清单。
检查清单：{task["checklist"]}
回复内容：{response}

返回 JSON：
{{
  "score": 0-10 的整数,
  "failures": ["失败模式 1", "失败模式 2"],
  "fix_hint": "应该改 prompt 的什么地方"
}}
不满足的项目作为 failures，给出针对性修改建议。"""

    msg = client.messages.create(
        model=MODEL,
        max_tokens=512,
        messages=[{"role": "user", "content": judge_prompt}],
    )
    try:
        data = json.loads(msg.content[0].text.strip())
        return data["score"], data["failures"], data["fix_hint"]
    except (json.JSONDecodeError, KeyError):
        return 0, [], ""

def evaluate(prompt, tasks):
    """在一组任务上跑完，返回平均分"""
    total = 0
    for task in tasks:
        resp = rollout(prompt, task)
        s, _, _ = score_with_critique(prompt, task, resp)
        total += s
    return total / len(tasks)

def collect_critiques(prompt, train_tasks, threshold=7):
    """收集训练集失败 case 的 critique，作为反馈信号"""
    critiques = []
    for task in train_tasks:
        resp = rollout(prompt, task)
        s, failures, fix_hint = score_with_critique(prompt, task, resp)
        if s < threshold:
            critiques.append({
                "input": task["input"],
                "response": resp,
                "failures": failures,
                "fix_hint": fix_hint,
            })
    return critiques

def propose_edit(prompt, critiques, rejected):
    """让 optimizer LLM 提一个改进版 prompt"""
    critique_text = "\n\n".join([
        f"输入: {c['input']}\n回复: {c['response']}\n"
        f"失败模式: {', '.join(c['failures'])}\n建议: {c['fix_hint']}"
        for c in critiques[:5]  # 用最差的 5 个
    ])
    rejected_text = "\n".join([f"- {r}" for r in rejected[-10:]])

    optimizer_prompt = f"""你是一个 prompt 的 optimizer。
当前 prompt：
---
{prompt}
---

下面是用这个 prompt 跑训练任务时失败的几个 case，每个都带失败模式和修改建议：
{critique_text}

下面这些方向之前试过，没用，不要再提：
{rejected_text}

请给出改写后的完整 prompt。要求：
1. 只改 1-2 处，不要整篇重写
2. 针对 critique 里提到的失败模式做精确修改
3. 不要用「视情况」「灵活处理」等模糊措辞
4. 直接输出新 prompt，不要解释"""

    msg = client.messages.create(
        model=MODEL,
        max_tokens=2048,
        messages=[{"role": "user", "content": optimizer_prompt}],
    )
    return msg.content[0].text

def train(prompt_path, train_path, val_path, num_steps=10, out_dir="outputs"):
    Path(out_dir).mkdir(exist_ok=True)
    prompt = Path(prompt_path).read_text()
    train_tasks = load_tasks(train_path)
    val_tasks = load_tasks(val_path)
    rejected = []
    history = []

    best_score = evaluate(prompt, val_tasks)
    best_prompt = prompt
    print(f"[step 0] baseline = {best_score:.2f}")
    history.append({"step": 0, "score": best_score, "action": "baseline"})

    for step in range(1, num_steps + 1):
        critiques = collect_critiques(best_prompt, train_tasks)
        if not critiques:
            print(f"[step {step}] 训练集已无失败 case，停止")
            break

        candidate = propose_edit(best_prompt, critiques, rejected)
        cand_score = evaluate(candidate, val_tasks)

        if cand_score > best_score:
            print(f"[step {step}] {best_score:.2f} -> {cand_score:.2f} ✓ keep")
            best_score = cand_score
            best_prompt = candidate
            history.append({"step": step, "score": cand_score, "action": "keep"})
            Path(f"{out_dir}/best_prompt.txt").write_text(best_prompt)
        else:
            print(f"[step {step}] {best_score:.2f} -> {cand_score:.2f} ✗ revert")
            rejected.append(candidate[:200])
            history.append({"step": step, "score": cand_score, "action": "revert"})

        # early stopping：连续 3 步不涨就停
        recent = history[-3:]
        if len(recent) == 3 and all(h["action"] == "revert" for h in recent):
            print(f"[step {step}] 连续 3 次 revert，停止")
            break

    Path(f"{out_dir}/history.json").write_text(json.dumps(history, indent=2))
    print(f"\n最终最优分: {best_score:.2f}")
    return best_prompt

if __name__ == "__main__":
    train("prompt.txt", "train_tasks.json", "val_tasks.json", num_steps=10)
```

### 跑起来

```bash
pip install anthropic
export ANTHROPIC_API_KEY="sk-ant-..."

# 准备 prompt.txt / train_tasks.json / val_tasks.json
# 任务格式：{"input": ..., "checklist": "需要 (1) X (2) Y (3) Z"}

python trainer.py
```

输出大概长这样：

```
[step 0] baseline = 5.20
[step 1] 5.20 -> 6.50 ✓ keep
[step 2] 6.50 -> 6.10 ✗ revert
[step 3] 6.50 -> 7.30 ✓ keep
[step 4] 7.30 -> 8.10 ✓ keep
[step 5] 8.10 -> 7.80 ✗ revert
[step 6] 8.10 -> 7.95 ✗ revert
[step 7] 8.10 -> 7.90 ✗ revert
[step 7] 连续 3 次 revert，停止

最终最优分: 8.10
```

### 这份代码做对了什么

把这条主线七个项目的核心机制都包进去了，删到最简：

- **结构化 critique 代替标量打分**：`score_with_critique` 返回 score + failures + fix_hint，不只是一个数（DSPy/MIPROv2
  之后的所有项目）
- **validation-gated edits**：训练集找改进方向，验证集打分裁决（DSPy 的 compile 范式 + GEPA 的 frontier 思路）
- **棘轮**：只在 `cand_score > best_score` 时 keep（autoresearch / SkillOpt 的 ratchet）
- **rejected buffer**：失败方向加入 `rejected`，下一轮告诉 optimizer 别再提（SkillOpt 的 negative replay）
- **限制单步改动**：propose_edit 的 prompt 里明确「只改 1-2 处」（GEPA 的 textual learning rate）
- **early stopping**：连续 3 步 revert 自动停（DSPy / GEPA 都有类似机制）

差距在哪：

- **没有 Pareto frontier**。GEPA 维护一个候选集合，每个候选在某些 task 上当前最强。这版只维护一个 best_prompt，遇到不兼容子分布会
  mode collapse。要扩 Pareto，把 evaluate 改成「每个 task 单独打分，存成向量」，candidate 入选条件改成「在至少一个 task 上严格优于现有
  frontier」。
- **judge 用的是同一个模型**。SkillLens 警告这有自评偏差。生产用的话 judge 换不同家模型。
- **没有 system-aware merge**。GEPA 会合并两个候选取长补短，这版没做。

跑过几轮就能感觉到，整套机制最关键的不是 optimizer 本身，是 critique 的质量。critique 准了，optimizer 自然能修对地方。critique
抓不到失败模式，optimizer 再强也是瞎改。

这是这条主线五年最重要的工程教训：**反馈通道的带宽决定了优化的上限。** 提案怎么提是次要的，反馈给得多准多结构化才是关键。

---

## 十、几个常见的坑

跑过几次大概会踩这些坑。

**坑 1：测试集太小**。5 个测试任务跑出来的方差太大。最少 15-20 个，覆盖训练集 + 验证集。

**坑 2：训练集和验证集污染**。验证集任务和训练集任务有重合时，验证分数会虚高。准备数据时严格分离。

**坑 3：judge 用同一个模型**。LLM 自评准确率约 46.4%（SkillLens 实测），和瞎猜没显著差异。生产用的话 judge 换不同家模型。

**坑 4：critique 写得太抽象**。让 judge 返回「这条 prompt 不够好」是没用的。要返回「这条 prompt
在涉及单位换算时一致失败，应该在指令里加一句『先把所有量统一到 SI 单位』」。critique 越具体，optimizer 越能命中。

**坑 5：让 optimizer 整篇重写**。改动幅度太大会破坏已经稳定的部分，分数大概率会掉。明确限制「只改 1-2 处」。

**坑 6：盯着 benchmark 分数，忽略真实任务分布**。验证集分数升了不一定意味着真实场景表现升了。一定要做线上 A/B。

---

## 十一、还没解决的问题

**多步 pipeline 的 credit assignment**。DSPy 优化的是多步 pipeline 的每个模块，但当某个 case 失败时，是哪一步该改？目前的方案是把
critique 撒给所有步骤，让 optimizer 自己判断。这在长 pipeline 上效果会衰减。神经网络靠链式法则解决这个问题，文本工件没有等价物。

**evaluator 的可靠性**。这条主线所有项目都假设有一个可靠的 evaluator/metric。但很多场景里 evaluator 本身就是 LLM judge，自带
46.4% 的不可靠性。GEPA 已经在试 meta-GEPA：让 evaluator 也参与进化。

**多目标优化的形式化**。GEPA 用 Pareto frontier 处理「在不同 task
上的不兼容偏好」，但实际生产中目标维度更多——准确率、成本、延迟、安全性、风格一致性。这些放到一起怎么权衡，目前还是工程直觉。

**和模型权重训练的边界**。Prompt 优化和 RLHF / SFT 在很多任务上效果接近，但成本差一个数量级。哪些能力应该放进权重，哪些应该放进
prompt？目前主要靠工程直觉，没有理论框架。一个朴素判断：高频、稳定、领域通用的能力放权重；低频、易变、特定场景的放
prompt。但这条边界还在移动。

---

## 十二、把 Prompt 当参数训练，意味着什么

回头看七个项目，最值得记住的不是某个具体技术，是一个工程范式的转变。

写 prompt 之前，**写**这件事被当成手艺。一句句调措辞、加例子、改格式，靠工程师的直觉判断好坏。

把 prompt 当参数训练之后，**写**这件事变成了**初始化**。手写一个 70 分的 baseline，相当于神经网络的 Xavier
initialization——不需要写得多好，只要不离最优解太远就行。剩下的工作交给训练循环：跑 rollout、收集 critique、生成候选、验证、保留改进。

这个转变在两件事上有现实意义。

**对个人开发者**：不再需要花两天调一段 prompt 到完美。花两小时写一份 70 分的 baseline，准备 30 个测试任务，用 DSPy 或 GEPA
跑一晚上，第二天起来就有一份接近 90 分的 prompt。优化的工作量外包给了优化循环。

**对组织**：prompt 库可以系统性管理，不靠几个工程师的个人经验。MIPROv2 给了一套可扩展的优化流程，GEPA 给了一套带多样性的进化框架，TextGrad
给了一套结构化反馈接口。这些工具让 prompt 工程从手艺变成可工程化的流程。

模型权重训练的工程纪律——验证集裁判、学习率控制、early stopping、batch optimization——花了二十年从研究走到生产。Prompt
训练在五年里走完了同样的路，只是被优化的对象从张量换成了文本，被替换的反向传播换成了 LLM 反馈。

GEPA 实测下来，文本反馈的样本效率比 RL 微调高 35 倍。Databricks 的实测里，开源模型 + GEPA 调出来的 prompt 在企业 agent 任务上比
Claude Opus 4.1 便宜 90 倍。这两个数字说明的同一件事：**模型权重在变成冻结的执行单元，优化的工作转移到外面那一层。**

下一个尺度是什么？看趋势，是从 prompt 长到 skill 文档（已经发生），从 skill 文档长到完整 agent 程序（ADAS
已经在做）。每一次单元变大，被替换的反向传播负担更重，但样本效率更高的优势也更明显。

你今天就可以开始跑。

---

## 参考

- Zhou, Y. et al. [Large Language Models Are Human-Level Prompt Engineers](https://arxiv.org/abs/2211.01910). arXiv:
  2211.01910, 2022. (APE)
- Yang, C. et al. [Large Language Models as Optimizers](https://arxiv.org/abs/2309.03409). arXiv:2309.03409, 2023. (
  OPRO)
- Fernando, C. et
  al. [Promptbreeder: Self-Referential Self-Improvement Via Prompt Evolution](https://arxiv.org/abs/2309.16797). arXiv:
  2309.16797, 2023.
- Khattab, O. et
  al. [DSPy: Compiling Declarative Language Model Calls into Self-Improving Pipelines](https://arxiv.org/abs/2310.03714).
  arXiv:2310.03714, 2023.
- Opsahl-Ong, K. et
  al. [Optimizing Instructions and Demonstrations for Multi-Stage Language Model Programs](https://arxiv.org/abs/2406.11695).
  arXiv:2406.11695, 2024. (MIPROv2)
- Yuksekgonul, M. et al. [TextGrad: Automatic "Differentiation" via Text](https://arxiv.org/abs/2406.07496). arXiv:
  2406.07496, 2024.
- Agarwal, E. et al. [PromptWizard: Task-Aware Prompt Optimization Framework](https://arxiv.org/abs/2405.18369). arXiv:
  2405.18369, 2024.
- Agrawal, L. A. et
  al. [GEPA: Reflective Prompt Evolution Can Outperform Reinforcement Learning](https://arxiv.org/abs/2507.19457).
  arXiv:2507.19457, 2025.
- [keirp/automatic_prompt_engineer](https://github.com/keirp/automatic_prompt_engineer). GitHub.
- [google-deepmind/opro](https://github.com/google-deepmind/opro). GitHub.
- [stanfordnlp/dspy](https://github.com/stanfordnlp/dspy). GitHub.
- [zou-group/textgrad](https://github.com/zou-group/textgrad). GitHub.
- [microsoft/PromptWizard](https://github.com/microsoft/PromptWizard). GitHub.
- [gepa-ai/gepa](https://github.com/gepa-ai/gepa). GitHub.
