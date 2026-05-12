# Content Analysis: draft.md

## Highlights & Key Insights

- **核心颠覆性数据**：同一个 Claude 3.5 Sonnet，Harness 升级带来 +16pt，模型换代只带来 +11pt——工程比模型代差更值钱
- **LangChain 数据**：仅改 Harness，TerminalBench 2.0 从第 30 名跳到第 5 名，模型权重一字未动
- **ReAct 关键数据**：纯 CoT 幻觉率 56% vs ReAct 约 0%——循环结构比模型更能决定幻觉率
- **工具数量悖论**：Vercel v0 砍掉 80% 工具，性能反而提升；Claude Code 懒加载实现 95% 上下文缩减
- **验证循环倍增效应**：Boris Cherny——给模型验证自己工作的方式，任务质量提升 2-3 倍
- **ACON 数据**：优先保留推理链而非工具输出，token 减少 26-54%，准确率维持 95%+
- **Von Neumann 比喻**："We have reinvented the Von Neumann architecture"——Harness 就是 Agent 的操作系统
- **Vivek Trivedy 金句**："If you're not the model, you're the harness."

## Structure Assessment

- 当前流：引子（数据冲击）→ 定义层（是什么）→ 组件层（12 个组件）→ 原则层（棘轮）→ 追踪层（完整循环）→ 决策层（7 个权衡）→ 对比层（框架）→
  实践层（落地代码）→ 趋势层（尾声）
- 结构非常清晰，层层递进。已有完整 ## 和 ### 层级
- 现有标题层级合理，无需调整

## Reader-Important Information

- 12 个组件的每一个都有工程实践价值
- Ralph Loop 模式是跨窗口长任务的可操作解法
- AGENTS.md 检查清单型写法 vs 风格指南型写法的对比极具实操价值
- 常见错误清单表格（10 项）是直接可用的检查工具
- 最小可运行骨架代码可直接复制使用

## Formatting Issues

- 文章无 YAML frontmatter，需要补全（title、slug、summary、description）
- 文章开头是 H1 标题，需提取至 frontmatter 后从正文移除
- 整体格式质量已经很高，粗体、列表、代码块使用得当
- 中英文间距可能有少量不规范处，需运行 autocorrect 脚本处理
- 少数 `**` 强调标记需要确认 CJK 边界是否正确（由脚本自动修复）

## Typos Found

None found.
