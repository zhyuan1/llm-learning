# 大纲 v2：Agent Harness 全解剖

文章目标：读完不需要再查任何东西。前半部分给决策者，后半部分给开发者。

主要资料：

- The-Anatomy-of-an-Agent-Harness.md（Akshay Pachaar，2026）← 核心综合来源
- agent-harness-engineering.md（Addy Osmani，2026）← 工程实践视角
- SWE-agent arXiv:2405.15793（Princeton，2024）
- ReAct arXiv:2210.03629（ICLR 2023）
- CoALA arXiv:2309.02427（2023）
- Anthropic: Building Effective Agents
- Chip Huyen: Agents（2025）
- Lilian Weng: LLM Powered Autonomous Agents

---

## 引子

悬念钩子：

- 同一个模型，3.8% vs 49%（SWE-bench）
- LangChain 只改 Harness 基础设施，从 TerminalBench 2.0 榜单 30 名开外跳到第 5
- "The problem isn't your model. It's everything around your model."（Addy Osmani）

**Harness Gap 概念**（Addy Osmani）：
> "The gap between what today's models can theoretically do and what you actually see them doing is largely a harness
> gap."
> 模型的理论能力和实际表现之间的距离，本质是 Harness 的距离。

---

## 第一部分：什么是 Harness（概念建立）

### 1.1 定义

三种等价的表述：

- Anthropic Claude Code 文档："the SDK is the agent harness that powers Claude Code"
- OpenAI Codex 团队：agent = harness（非模型的全部基础设施）
- LangChain Vivek Trivedy 的公式：**"If you're not the model, you're the harness."**

区分容易混淆的三个层：

- Agent = 涌现行为（目标导向、使用工具、自我纠错的实体）
- Harness = 产生这个行为的机器
- "当有人说'我搭了一个 Agent'，他的意思是他搭了一个 Harness，然后把它对准了一个模型"

### 1.2 冯·诺依曼类比

Beren Millidge（2023）："We have reinvented the Von Neumann architecture"

| 计算机组件    | Agent 等价物  |
|----------|------------|
| CPU      | LLM 本身     |
| RAM（快但小） | 上下文窗口      |
| 磁盘（大但慢）  | 外部数据库/向量存储 |
| 设备驱动     | 工具集成       |
| 操作系统     | Harness    |

### 1.3 三层工程的区别

- Prompt 工程：给模型看什么指令
- Context 工程：管理模型在什么时候看到什么
- **Harness 工程**：包含以上两者，加上整个应用基础设施（工具编排、状态持久化、错误恢复、验证循环、安全执行、生命周期管理）

---

## 第二部分：Harness 的 12 个组件

*（基于 Akshay Pachaar 的综合框架，补充各来源实证数据）*

### 2.1 编排循环（Orchestration Loop）

- TAO 循环（Thought-Action-Observation）= ReAct 循环的落地形式
- Anthropic 的表述："dumb loop"——循环本身很简单，所有智能在模型里
- 机械上通常就是一个 while 循环，复杂度在循环管理的东西里
- 终止条件层：无工具调用 / 达到最大轮次 / token 预算耗尽 / 安全触发 / 用户中断
- 一个简单问题：1-2 轮；复杂重构：数十次工具调用跨多轮

**实证**：ReAct 让 HotpotQA 幻觉率从 56% → ~0%（Yao et al. 2023）——循环结构决定幻觉率，不是模型智力

### 2.2 工具层（Tools）

- 工具 = 模型的"手"，以 schema（名称、描述、参数类型）形式注入上下文
- 工具层负责：注册、schema 验证、参数提取、沙箱执行、结果捕获、格式化为可读 Observation

**Claude Code 工具六类**：文件操作、搜索、执行、Web 访问、代码智能、子 Agent 生成
**OpenAI Agents SDK 工具三类**：function tools、hosted tools（WebSearch/CodeInterpreter/FileSearch）、MCP 工具

**关键设计原则**（与 Anthropic 建议一致）：

- 工具描述 = 给初级开发者写的文档，包含示例、边界、参数约束
- Poka-yoke：绝对路径消灭了相对路径错误
- 工具数量悖论：Vercel 删掉 v0 80% 的工具，反而得到更好结果；Claude Code 靠懒加载实现 95% 上下文缩减
- 阈值：~10 个重叠工具是考虑拆多 Agent 的信号
- **渐进披露（Progressive Disclosure）**（Addy Osmani）：启动时不加载全量工具和指令，仅在任务明确需要时才暴露——减少噪音，保护上下文
- 安全风险：未经验证的 MCP 服务器的工具描述会注入进 prompt，形成提示注入攻击面

### 2.3 记忆（Memory）

三个时间维度：

- **短期记忆**：单次会话内的对话历史（上下文窗口 = RAM）
- **长期记忆**：跨会话持久化
    - Anthropic：CLAUDE.md 项目文件 + 自动生成的 MEMORY.md
    - LangGraph：命名空间组织的 JSON Store
    - OpenAI：Sessions（SQLite 或 Redis 支持）

**Claude Code 三层层级**：

1. 轻量索引（~150 字符/条目，始终加载）
2. 详细主题文件（按需拉取）
3. 原始记录（仅通过搜索访问）

**设计原则**：Agent 把自己的记忆当作"提示"，在行动前验证实际状态

### 2.4 上下文管理（Context Management）

**核心问题：上下文腐化（Context Rot）**

- "Lost in the Middle"（Stanford）：关键内容落在窗口中间时，性能下降 30%+
- 即使是百万 token 窗口，随着上下文增长，指令跟随能力依然衰减

**五种生产级策略**：

1. **压缩（Compaction）**：历史摘要化，保留架构决策和未解决 bug，丢弃冗余工具输出
2. **观察遮蔽（Observation Masking）**：JetBrains Junie 隐藏旧工具输出，但保留工具调用可见
3. **即时检索（JIT Retrieval）**：Claude Code 用 grep/glob/head/tail，而非加载完整文件
4. **工具调用卸载（Tool-call Offloading）**（Addy Osmani）：2000 行日志写入文件系统，上下文只保留关键头尾
5. **渐进披露 + 子 Agent 委托**：按需加载 + 每个子 Agent 只返回 1000-2000 token 压缩摘要

**量化证据**：ACON 研究——优先保留推理链而非原始工具输出，token 减少 26-54%，准确率保持 95%+

### 2.5 Prompt 构建（Prompt Construction）

层次化结构（从高优先级到低）：

1. 系统提示（服务端控制）
2. 工具定义
3. 记忆文件
4. 对话历史
5. 当前用户消息

**定位原则**：重要上下文放在 prompt 的开头和结尾（"Lost in the Middle" 的推论）

### 2.6 输出解析（Output Parsing）

现代做法：原生工具调用（结构化 tool_calls 对象，不需要解析自由文本）

- 有工具调用 → 执行并循环
- 没有工具调用 → 最终答案

结构化输出：OpenAI/LangChain 通过 Pydantic 模型约束 schema
兜底：RetryWithErrorOutputParser（把原始 prompt + 失败结果 + 解析错误喂回模型）

### 2.7 状态管理（State Management）

- **LangGraph**：类型化字典 + reducer 合并更新，super-step 边界 checkpoint（支持恢复和时间旅行调试）
- **OpenAI**：四种互斥策略（应用内存 / SDK Sessions / 服务端 Conversations API / previous_response_id 链）
- **Claude Code**：git commit 作检查点 + 进度文件作结构化草稿本

### 2.8 错误处理（Error Handling）

**复利效应**：99% 单步成功率 × 10 步 = 90.4% 任务完成率（0.99^10）

**LangGraph 四种错误分类**：

- 瞬时错误（指数退避重试）
- LLM 可恢复（作为 ToolMessage 返回，让模型调整）
- 用户可修复（中断等待人工输入）
- 意外错误（向上冒泡调试）

Anthropic 做法：在工具处理器内捕获失败，作为错误结果返回，保持循环运行
Stripe 生产实践：最多重试 2 次

### 2.9 护栏与安全（Guardrails & Safety）

**Hooks = 执行层**（Addy Osmani 的核心概念）：
> "Hooks bridge the gap between requesting an action and enforcing it."

Hooks 在特定生命周期触发：工具调用前 / 文件编辑后 / commit 前

- 阻断破坏性命令
- 强制自动格式化（节省 token）
- 触发测试套件

**黄金原则**：成功沉默，失败冗长（Silent on success, verbose on failure）

- 类型检查通过 → Agent 听不到任何东西
- 类型检查失败 → 错误信息直接注入循环，触发自我纠正

**OpenAI SDK 三层**：

- 输入护栏（作用于首个 Agent）
- 输出护栏（作用于最终输出）
- 工具护栏（每次工具调用都触发）
- "Tripwire" 机制：触发即立刻停止 Agent

**Anthropic 架构原则**：权限执行与模型推理在架构上分离

- 模型决定"试图做什么"
- 工具系统决定"是否被允许"
- Claude Code 独立管控约 40 个工具能力，三阶段：信任建立 → 权限检查 → 高风险操作显式确认

### 2.10 验证循环（Verification Loops）

**把 Demo 和生产级 Agent 分开的关键组件**

Anthropic 推荐三种验证方式：

1. 基于规则（测试、Linter、类型检查）→ 确定性真实
2. 视觉反馈（Playwright 截图）→ UI 任务
3. LLM-as-Judge（独立子 Agent 评估）→ 语义问题

**Boris Cherny（Claude Code 作者）**：给模型一种验证自己工作的方式，质量提升 2-3x

Martin Fowler 框架：

- Guides（前馈）：行动前引导
- Sensors（反馈）：行动后观察

### 2.11 子 Agent 编排（Subagent Orchestration）

**Claude Code 三种执行模型**：

- Fork（字节级完整复制父上下文）
- Teammate（独立终端面板，文件信箱通信）
- Worktree（独立 git worktree，每个 Agent 有独立分支）

**OpenAI SDK**：agents-as-tools（有界子任务）vs handoffs（完全转交控制）

**何时拆多 Agent**（Anthropic + OpenAI 共识）：先最大化单 Agent，仅在工具重叠超过 ~10 个或存在明确分离任务域时才拆分

---

## 第三部分半：Harness 工程的核心心法——棘轮原则

（Addy Osmani 独有，值得单独成节）

**The Ratchet（棘轮原则）**：每次失败 → 永久规则，Harness 只进不退

> "Whenever an agent fails, you engineer a permanent solution so it never makes that exact mistake again."

具体示例链：

1. Agent 提交了一个注释掉的测试被合并了
2. AGENTS.md 增加规则："不得注释掉测试，必须删除或修复"
3. pre-commit hook 自动标记 diff 中的 `.skip(`
4. Reviewer 子 Agent 更新为阻止注释掉的测试

**AGENTS.md 的正确使用方式**：

- 不是风格指南，是飞行员检查清单
- 每一行规则都必须能追溯到一次具体的历史失败
- 保持简短；没有历史根基的规则会稀释重要规则的权重

**实践推论**：

- 每个团队的正确 Harness 都不一样——它由这个团队的独特失败历史塑造
- 这是 Harness 工程作为一门独立工艺（而非通用框架）存在的原因

---

7 个步骤（基于文章中的 Step-by-Step Walkthrough）：

1. **Prompt 组装**：系统提示 + 工具 schema + 记忆文件 + 对话历史 + 当前消息（关键内容置首尾）
2. **LLM 推理**：输入送模型，产出文本 / 工具调用请求 / 两者混合
3. **输出分类**：纯文本 → 结束；工具调用 → 执行；Handoff 请求 → 切换 Agent 重启
4. **工具执行**：验证参数 → 权限检查 → 沙箱执行 → 捕获结果（只读可并发，写操作串行）
5. **结果打包**：格式化为 LLM 可读消息；错误作为错误结果返回让模型自校正
6. **上下文更新**：追加历史；接近窗口上限时触发压缩
7. **循环**：回到步骤 1，直到满足终止条件

---

## 第四部分：七个关键架构决策

（Akshay Pachaar 框架，逐一带实证数据）

1. **单 Agent vs 多 Agent**：先压榨单 Agent；工具重叠 >10 或明确域分离再拆
2. **ReAct vs 计划-执行**：ReAct 灵活，每步成本高；LLMCompiler 报告 3.6x 加速 vs 顺序 ReAct
3. **上下文管理策略**：五种（见 2.4）
4. **验证循环设计**：计算验证（确定性）vs 推理验证（语义）
5. **权限与安全架构**：宽松（快但有风险）vs 严格（安全但慢）
6. **工具范围策略**：最小工具集原则；每步按需暴露
7. **Harness 厚度**：Anthropic 押注薄 Harness + 模型进化；LangGraph 等 graph-based 框架押注显式控制

---

## 第五部分：各框架实现对比（开发者参考）

| 框架                | 循环模型                                 | 状态管理                   | 工具模型                | 特色             |
|-------------------|--------------------------------------|------------------------|---------------------|----------------|
| Anthropic SDK     | query() 异步迭代器                        | git + 进度文件             | 6 类工具 + MCP         | 薄 Harness，模型主导 |
| OpenAI Agents SDK | Runner（async/sync/stream）            | Sessions/Conversations | function/hosted/MCP | 代码优先，Python 原生 |
| LangGraph         | state graph（node + conditional edge） | super-step checkpoint  | graph node          | 显式控制流，可调试      |
| CrewAI            | 角色制多 Agent                           | Crew 级状态               | 按角色分配               | 协作导向           |
| AutoGen           | 对话驱动                                 | Magentic 任务台账          | 五种编排模式              | 群聊 + 动态协调      |

---

## 第六部分：落地实践

### 6.1 最小可运行 Harness（伪代码骨架）

```python
history = []
while True:
    prompt = assemble(system, tools, memory, history, user_msg)
    output  = llm(prompt)
    if not output.tool_calls:
        return output.text          # 终止
    for call in output.tool_calls:
        result = execute(call)      # 沙箱执行
        history.append(call + result)
    if len(history) > MAX_TURNS:
        break                       # 安全停止
```

### 6.2 工具定义的黄金标准

好的工具定义包含：

- 名称（动词 + 名词）
- 一句话描述（用途，不是实现）
- 参数 schema（类型 + 约束 + 枚举值）
- 使用示例（1-2 个）
- 错误行为说明（失败时返回什么）
- 与相邻工具的边界（何时用这个而不是那个）

### 6.3 常见错误清单

| 错误       | 后果                  | 修复                      |
|----------|---------------------|-------------------------|
| 工具描述太短   | 模型猜测参数含义 → 参数错误     | 写完整的 docstring          |
| 工具数量超载   | 误选率上升，性能下降          | 动态加载，每步按需暴露             |
| 使用相对路径   | 切换目录后路径失效           | 强制绝对路径                  |
| 空输出沉默    | 模型误判为成功             | 显式返回 "No results found" |
| 无终止条件    | 无限循环 + 成本失控         | 最大轮次 + token 预算双重保险     |
| 错误信息不结构化 | 模型无法定位问题，无法 recover | 返回结构化错误（类型 + 位置 + 建议）   |
| 跨会话无状态   | 每次从零开始              | 进度文件 + 检查点              |

### 6.4 长任务跨窗口：Ralph Loop 模式

Anthropic 的两阶段模式：

1. **初始化 Agent**：建环境、写进度文件、功能列表、初始 git commit
2. **编码 Agent**（每个后续会话）：读 git log + 进度文件定向 → 选最高优先级未完成任务 → 执行 → commit → 写摘要

文件系统提供跨上下文窗口的连续性。

---

## 尾声

**Harness 不会消失，只会迁移**（Addy Osmani）：
> "As models improve, the need for a harness doesn't disappear — it shifts."

模型进化让"上下文焦虑"类缓解措施变得多余，但同时打开了以前不可能的任务，带来全新的失败模式。每个 Harness
组件都编码了一个假设："模型单独做不到这件事"——模型进化后，过时的脚手架拆除，新的脚手架覆盖更高处。

**Harness-as-a-Service（HaaS）的行业趋势**：
行业正在从"基于 LLM API（提供补全）"转向"基于 Harness API（提供运行时）"。现代 SDK
直接提供循环、工具、上下文管理、Hooks、沙箱——从零搭建编排已经变成了配置一个设计良好的 Harness 框架。

**Harness → 编译器的未来**：
> "Harnesses stop being static configuration files and start acting much more like compilers."

开放问题：多 Agent 并行编排、Agent 分析自己的执行轨迹来修复 Harness 层的故障、动态即时组装工具的环境。

**未来-proof 测试**（Akshay Pachaar）：换用更强的模型，性能提升，但不需要增加 Harness 复杂度——这就是设计合理的信号。

回到开头：你的 Agent 失败时，不要怪模型，看 Harness。

---

## Further Reading（6 个最值得深读）

1. **The Anatomy of an Agent Harness**（Akshay Pachaar，2026）— 最全面的综合框架 ★ 推荐起点
2. **Harness Engineering**（Addy Osmani，2026）— 工程实践与棘轮原则
3. **SWE-agent**（arXiv:2405.15793，Princeton，2024）— ACI 设计的最强实验证据
4. **Anthropic: Building Effective Agents** — 最实用的工程原则
5. **ReAct**（arXiv:2210.03629，ICLR 2023）— 执行循环的理论基础
6. **CoALA**（arXiv:2309.02427，2023）— 最完整的概念框架和词汇表
