---
illustration: 03
type: flowchart
style: blueprint
palette: default
language: zh
aspect: 16:9
article: agent-long-task/draft.md
section: 四、模型需要主动写给自己看
---

# Prompt: 03-flowchart-scratchpad-workflow

Blueprint-style technical flowchart showing how Scratchpad mechanism works in AI Agent long-horizon tasks. Dark navy
schematic aesthetic.

## LAYOUT

Central horizontal process flow (left to right) with a persistent Scratchpad panel below, and benefit annotations on the
right.

## ZONES

### HEADER

- Title: "Scratchpad 工作机制"
- Subtitle: "Agent 的显式外化记忆 — YC-Bench 最强预测变量"
- Style: white monospace, top-centered

### MAIN FLOW (horizontal, left to right)

Step boxes connected by arrows:

1. **步骤①**: "接收任务/子任务"
    - Box: white outline, small robot/agent icon

2. **步骤②**: "关键决策点前 → 写状态评估"
    - Box: cyan outline, pencil/write icon
    - Downward arrow to SCRATCHPAD with label "写入"
    - Note: "当前状态·假设·风险"

3. **步骤③**: "调用工具"
    - Box: white outline, API/tool icon
    - Arrow right

4. **步骤④**: "工具返回结果"
    - Box: white outline, return arrow icon

5. **步骤⑤**: "写解读 + 修正计划"
    - Box: cyan outline, pencil icon
    - Downward arrow to SCRATCHPAD with label "更新"
    - Note: "结果解读·计划修正"

6. **步骤⑥**: "引用 Scratchpad 继续推理"
    - Box: bright cyan fill, reference/link icon
    - Upward arrow FROM SCRATCHPAD with label "明确引用"

7. **步骤⑦**: "完成 or 下一子任务"
    - Box: white outline, checkmark icon

Arrows: thick white with arrowheads, label above each key arrow

### SCRATCHPAD PANEL (horizontal bar below main flow)

- Persistent horizontal panel spanning the width
- Label text (plain text only, no emoji or symbols): "Scratchpad（显式中间笔记）"
- Interior: shows text snippet examples in monospace:
    - "当前假设：异常值集中于时间维度..."
    - "工具调用#20返回：分布模式与假设不符 → 修正计划..."
- Visual: notebook/document aesthetic with cyan border, slightly lighter navy background

### RIGHT ANNOTATIONS PANEL

Three benefit boxes stacked vertically, each starting with a plain checkmark glyph "v" or a small green tick drawn as
part of the box design:

1. "绕过注意力稀释"
2. "迫使清晰状态整理"
3. "可被后续步骤明确引用"

- Style: small white text boxes with cyan tick marks drawn as simple line-art, blueprint card style

### BOTTOM COMPARISON

- Contrast strip:
    - Left side: dim gray box labeled "依赖内部记忆" with a small red X icon (small, same size as the text, drawn as two
      short crossing lines, not oversized)
    - Right side: bright cyan box labeled "显式外化记忆" with a simple checkmark drawn as line-art
    - Center arrow pointing right

## STYLE RULES

- Background: dark navy blue, blueprint grid — no hex codes or color values anywhere in the image
- Step boxes: white outline, with cyan highlight for write/reference steps
- Scratchpad panel: distinct lighter navy background, cyan border
- Active steps: bright cyan outlines
- Annotations: dim but readable
- All text: white or light cyan, monospace — plain text only, absolutely no emoji characters
- Flow arrows: thick white with technical arrowhead style
- Aspect ratio: 16:9
