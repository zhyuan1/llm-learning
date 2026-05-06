---
type: infographic / framework / flowchart / comparison
density: balanced (4 images)
style: blueprint
palette: default (blueprint colors)
language: zh
image_count: 4
---

# Article Illustration Outline

Article: agent-long-task/draft.md
Topic: AI Agent 为什么跑着跑着就不对了（上下文锁定问题）

---

## Illustration 1

**Position**: 第一章末尾（"一、两组数据"章节之后）
**Type**: infographic
**Purpose**: 用直观的数据对比图展示 TravelPlanner 0.6% 和 UltraHorizon 的失败数据，让读者直接感受到问题的严重性
**Visual Content**: 两组对比数据——左侧 TravelPlanner：GPT-4 通过率 0.6%（vs 感知预期 60%）；右侧 UltraHorizon：人类得分 26.52
vs Agent 得分 14.33；底部：56% 失败归因于上下文管理失败
**Filename**: 01-infographic-benchmark-data.png

---

## Illustration 2

**Position**: 第三章末尾（"三、上下文锁定"章节之后）
**Type**: framework
**Purpose**: 可视化"上下文锁定"的两个底层机制：注意力稀释 + 路径依赖，以及两者叠加后的锁定效应
**Visual Content**: 双机制框架图——上方：Transformer 注意力稀释示意（token 越多，早期 token
注意力越薄）；下方：路径依赖链（早期错误假设 → 约束后续生成 → 锁定效应）；中间：两者叠加 = 上下文锁定；右侧注解：上下文达到窗口
40% 时性能断崖
**Filename**: 02-framework-context-locking.png

---

## Illustration 3

**Position**: 第四章末尾（"四、模型需要主动写给自己看"章节之后）
**Type**: flowchart
**Purpose**: 展示 Scratchpad 机制的工作流：在每个关键决策点前写状态、工具调用后写解读，形成"显式外化 → 可靠引用"的正向循环
**Visual Content**: 工作流程图——步骤1：关键决策点前写状态评估（→ scratchpad）；步骤2：工具调用；步骤3：结果返回后写解读和计划修正（→
scratchpad）；步骤4：后续步骤明确引用 scratchpad；旁注：绕过注意力稀释 / 迫使状态整理 / YC-Bench 最强预测变量
**Filename**: 03-flowchart-scratchpad-workflow.png

---

## Illustration 4

**Position**: 第五章末尾（"五、现有解法的三个方向"章节之后）
**Type**: comparison
**Purpose**: 三种解决方案横向对比，帮助读者快速理解各方案的思路差异和适用场景
**Visual Content**: 三栏对比——Context-Folding（主动折叠：分支上下文 → 压缩摘要写回主线，32K token 达到 62% BrowseComp）/
ACON（任务感知压缩：历史>4096 tokens 触发 + 观察>1024 tokens 触发，26-54% token 压缩）/ MemAgent（分段覆写：RL 训练，8K 训练但处理
3.5M tokens，95%+ 准确率）；底部共同点：都是主动管理上下文，不依赖更大窗口
**Filename**: 04-comparison-three-solutions.png
