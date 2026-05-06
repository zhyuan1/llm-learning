---
illustration: 04
type: comparison
style: blueprint
palette: default
language: zh
aspect: 16:9
article: agent-long-task/draft.md
section: 五、现有解法的三个方向
---

# Prompt: 04-comparison-three-solutions

Blueprint-style technical comparison diagram of three context management solutions for AI Agents. Three-column layout on
dark navy background.

## LAYOUT

Three equal vertical columns, each representing one solution, with a shared header and a unified bottom summary bar.

## ZONES

### HEADER

- Title: "三种上下文管理方案"
- Subtitle: "都不依赖更大的上下文窗口"
- Style: white monospace bold, top-centered, with cyan underline separator

### COLUMN 1: Context-Folding

**Top label**: "Context-Folding"
**Subtitle**: "主动分支折叠"
**Icon area**: diagram showing branching context with fold/collapse arrow

**Mechanism visual** (mini flowchart inside column):

- Box: "进入子任务分支上下文"
- Arrow down: "完成子任务"
- Box: "折叠 → 核心发现摘要"
- Arrow down (thick, cyan): "写回主线上下文"
- Box (dim, crossed): "丢弃中间步骤"

**Key metrics**:

- "32K token 预算"
- "BrowseComp-Plus: 62.0%"
- "SWE-Bench Verified: 58.0%"
- "对比基线需要 327K token"

**Tag**: "上下文效率: ~10x" in cyan badge

---

### COLUMN 2: ACON (Microsoft Research)

**Top label**: "ACON"
**Subtitle**: "任务感知压缩"
**Attribution**: "Microsoft Research"
**Icon area**: funnel/compression diagram

**Mechanism visual**:

- Two trigger boxes:
    - "历史 > 4096 tokens → 历史压缩"
    - "观察 > 1024 tokens → 观察压缩"
- Arrow down: "任务感知 Prompt 选择性保留"
- Result box: "精简高密度上下文"

**Key metrics**:

- "Token 压缩: 26-54%"
- "3个 Benchmark 维持或提升完成率"
- "中等模型(14B) → 接近大模型表现"

**Tag**: "兼容现有框架" in cyan badge

---

### COLUMN 3: MemAgent

**Top label**: "MemAgent"
**Subtitle**: "分段覆写记忆"
**Icon area**: sliding window with overwrite arrow

**Mechanism visual**:

- "文档分段处理"
- Arrow: "处理一段"
- Decision box: "RL训练决策：哪些值得写入记忆？"
- Arrow: "覆写记忆状态"
- Arrow: "处理下一段"

**Key metrics**:

- "训练: 8K 上下文"
- "处理: 3.5M tokens (稳定)"
- "性能损失: < 5%"
- "512K RULER: 95%+ 准确率"

**Tag**: "研究成果 (非生产)" in dim badge

---

### COLUMN DIVIDERS

- Thin cyan vertical lines between columns
- Blueprint schematic aesthetic

### BOTTOM SUMMARY BAR

- Full-width, slightly elevated background
- Central unified insight:
  "共同思路：上下文管理是主动决策，不是被动等待窗口更大"
- Three small icons below each column's key metric for quick reference
- Border: cyan top line

## STYLE RULES

- Background: dark navy (#0a1628), blueprint grid
- Column headers: white (#ffffff) bold
- Key metrics: cyan (#00d4ff) numbers
- Mechanism visuals: white outline boxes with thin cyan connectors
- Tags/badges: cyan background, dark text
- Dim badge: gray background
- Column dividers: cyan thin lines
- Bottom bar: slightly lighter navy, cyan top border
- All text: white or light cyan, monospace feel
- Aspect ratio: 16:9
