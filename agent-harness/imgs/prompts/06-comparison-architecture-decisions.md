---
type: comparison
style: notion
palette: default
aspect: 16:9
language: zh
---

# Prompt

Notion-style minimalist hand-drawn comparison table. White background, black ink, blue accent.

ZONES:

- TOP (10%): Title: "七个关键架构决策" / subtitle: "每个决策都有明确的工程权衡"
- MAIN TABLE (80%): 7-row comparison grid with 4 columns
    - Column headers (bold, bottom-bordered): "决策" | "选项 A" | "选项 B" | "核心信号"
    - Row 1: "单 vs 多 Agent" | "先压榨单Agent极限" | "专业分工明确时拆分" | "工具重叠 > 10 个？"
    - Row 2: "ReAct vs 计划执行" | "开放任务动态调整" | "可预测任务并发提速 3.6×" | "步骤可预测？"
    - Row 3: "上下文策略" | "< 20步全量保留" | "> 100步子Agent委托" | "任务规模？"
    - Row 4: "验证循环" | "确定性优先（测试/Lint）" | "语义验证（LLM-as-Judge）" | "贵的放后面"
    - Row 5: "权限架构" | "只读自动批准" | "破坏性操作显式审批" | "操作类型分类"
    - Row 6: "工具范围" | "渐进披露当前所需" | "避免全量加载" | "Vercel: 砍80%性能↑"
    - Row 7 (blue highlight row): "Harness 厚度" | "薄：模型主导" | "厚：显式控制流" | "换模型不改Harness？"
    - Alternating row background: odd rows white, even rows very light gray wash
- BOTTOM (10%): One callout box: "换用更强模型性能提升但无需修改 Harness → Harness 设计合理"

LABELS: All Chinese. All 7 row labels and all cell content as specified.

COLORS: White/light-gray alternating rows, black ink for text and borders, blue (#6B9FD4) only for Row 7 and the bottom
callout.

STYLE: Notion hand-drawn. Table borders are hand-sketched lines. Header row has bold bottom border. Clean alignment.

ASPECT: 16:9 wide (table fits best in wide format)
