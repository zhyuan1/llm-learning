---
type: mixed
density: per-section
style: notion
palette: default
image_count: 6
language: zh
---

## Illustration 1

**Position**: 引子（同一个模型，为什么差距超过 10 倍）
**Purpose**: 用数字冲击力直观呈现 Harness Gap —— 相同模型、不同基础设施，带来倍数级的性能差异
**Type**: infographic
**Visual Content**: 对比条形图 + 关键数字。三行对比：Claude 3.5 Sonnet + 普通聊天界面 = 33%；Claude 3.5 Sonnet + Harness =
49%；Claude 3 Opus + Harness = 22%。核心标注：「Harness 升级 +16pt」vs「模型升代 +11pt」。底部一行大字：工程比模型代差更值钱。
**Filename**: 01-infographic-harness-gap.png

---

## Illustration 2

**Position**: 第一部分：Harness 是什么 —— Von Neumann 比喻 + 三层工程区别
**Purpose**: 建立概念框架，帮助读者理解 Harness 在整体架构中的位置
**Type**: framework
**Visual Content**: 两个并排框架。左框：经典计算机分层（CPU / RAM / 磁盘 / 设备驱动 / 操作系统），右框：Agent 等价分层（LLM /
上下文窗口 / 外部存储 / 工具层 / Harness）。连接箭头显示对应关系。框架下方三列横向对比：Prompt 工程 → Context 工程 → Harness
工程，每列一行说明其作用范围。
**Filename**: 02-framework-von-neumann.png

---

## Illustration 3

**Position**: 第二部分组件一：编排循环
**Purpose**: 可视化 ReAct 执行循环的运作机制，以及推理与行动交织为何能降低幻觉率
**Type**: flowchart
**Visual Content**: 圆形循环图。节点顺序：组装 Prompt → LLM 推理（Thought）→ 解析输出 → 工具执行（Action）→ 结果返回（Observation）→
更新历史 → 回到组装 Prompt。循环外侧标注终止条件：无工具调用 / 最大轮次 / Token 耗尽 / 安全触发。右下角对比小图：纯 CoT 幻觉率
56% vs ReAct 幻觉率 ~0%。
**Filename**: 03-flowchart-react-loop.png

---

## Illustration 4

**Position**: 第二部分组件四：上下文管理
**Purpose**: 让读者直观理解上下文腐化问题，以及五种管理策略各自的作用
**Type**: infographic
**Visual Content**: 上半部分：一个上下文窗口示意图，左边标注「关键指令在开头/结尾有效」，中间标注「信息落中间 → 准确率下降
30%」（Lost in the Middle）。下半部分：五格横排，每格一个策略图标 + 名称 + 一句话说明：压缩（折叠历史）/ 观察遮蔽（隐藏输出保留调用记录）/
即时检索（按需拉取）/ 工具调用卸载（大输出写文件）/ 子Agent委托（独立窗口深探索）。底部数据：token 减少 26-54%，准确率维持 95%+。
**Filename**: 04-infographic-context-management.png

---

## Illustration 5

**Position**: 第三部分：棘轮原则
**Purpose**: 可视化「每次失败 → 永久修复」的三层递进机制
**Type**: framework
**Visual Content**: 棘轮齿轮图。中心是一个棘轮，旁边三条向上的箭头代表三层修复。第一层（最外）：Prompt 层 → AGENTS.md
添加规则。第二层：Hook 层 → pre-commit 脚本确定性阻断。第三层（最内）：验证层 → Reviewer 子Agent评审标准更新。左侧失败案例：「PR
合并了注释掉的测试」，右侧结果：「该错误永久从系统中消失」。底部标注：Prompt 是概率性的；Hook 是确定性的。
**Filename**: 05-framework-ratchet.png

---

## Illustration 6

**Position**: 第五部分：七个关键架构决策
**Purpose**: 给读者一张可参考的架构决策地图，帮助快速定位每个决策的核心权衡
**Type**: comparison
**Visual Content**: 7 行决策对比表。每行：决策名 | 选项A | 选项B | 核心权衡信号。内容：单Agent vs 多Agent（工具重叠<10 vs >
10）/ ReAct vs 计划执行（开放任务 vs 可预测任务）/ 上下文策略（<20步全量 vs >100步子Agent委托）/ 验证循环（确定性优先 vs
语义完备）/ 权限架构（宽松自动 vs 严格审批）/ 工具范围（全量加载 vs 渐进披露）/ Harness厚度（薄模型主导 vs 厚显式控制流）。
**Filename**: 06-comparison-architecture-decisions.png
