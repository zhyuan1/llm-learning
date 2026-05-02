---
type: flowchart
style: minimal-flat
palette: default-soft
language: zh
aspect: 16:9
output: ../03-flowchart-openclaw-memory.png
---

Create a clean minimal-flat flowchart explaining OpenClaw memory design for a Chinese technical article.

TITLE: “OpenClaw：让记忆在回复前浮现”

MAIN FLOW (top row):

1. “用户消息” → 2. “Active Memory 子 agent” → 3. “相关记忆注入上下文” → 4. “主 agent 回复”

Add a note under step 2:
“先召回，再回答”

MEMORY LAYERS (bottom row):

- “memory/YYYY-MM-DD.md：短期日记”
- “memory flush：压缩前抢救”
- “DREAMS.md：候选复查”
- “MEMORY.md：长期记忆”
  Connect them with arrows showing promotion: 短期信号 → 筛选 / dreaming → 长期记忆.

SIDE ANNOTATIONS:

- “召回时机 > 单纯存储”
- “短期痕迹需要晋升机制”
- “Markdown 文件可审查、可回滚”

STYLE:

- Minimal flat vector flowchart.
- Use warm orange highlight for Active Memory.
- Use calm blue for files, green for promotion arrows.
- Rounded boxes, thin arrows, clear spacing.

TYPOGRAPHY:
Chinese labels must be large and readable.

ASPECT:
16:9, suitable for WeChat article.
