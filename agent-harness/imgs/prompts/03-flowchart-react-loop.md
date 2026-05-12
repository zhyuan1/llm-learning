---
type: flowchart
style: notion
palette: default
aspect: 1:1
language: zh
---

# Prompt

Notion-style minimalist hand-drawn line art flowchart. White background, black ink, sparse blue accent.

ZONES:

- MAIN ZONE (70%): Circular loop diagram in the center, clockwise flow with 6 nodes connected by curved arrows
    - Node 1 (top): "组装 Prompt" (rectangle)
    - Node 2 (top-right): "LLM 推理 Thought" (rounded rectangle, soft blue fill)
    - Node 3 (right): "解析输出" (diamond shape — decision)
    - Node 4 (bottom-right): "工具执行 Action" (rectangle)
    - Node 5 (bottom): "结果返回 Observation" (rectangle)
    - Node 6 (bottom-left): "更新历史" (rectangle)
    - Arrow from Node 6 curves back up to Node 1 completing the loop
    - From Node 3 diamond: two branches — "有工具调用 →" continues to Node 4; "无工具调用 →" exits right with label "
      最终答案 ✓"
- RIGHT SIDE (15%): Termination conditions box, hand-drawn dashed border
    - Title: "终止条件"
    - Bullet list:
        - 无工具调用
        - 最大轮次超限
        - Token 预算耗尽
        - 安全护栏触发
        - 用户中断
- BOTTOM ZONE (15%): Small side-by-side comparison strip
    - Left cell: "纯 CoT" / large number "56%" / label "幻觉率"
    - Right cell (highlighted blue): "ReAct" / large number "~0%" / label "幻觉率"
    - Caption: "推理与行动交织 → 每步外部 grounding"

LABELS: All Chinese. Key labels on nodes visible. Numbers 56% and ~0% prominent.

COLORS: White background, black ink. Soft blue only for "LLM 推理" node and the ReAct comparison cell.

STYLE: Notion hand-drawn. Slight sketch texture. Clean flow with arrows showing direction.

ASPECT: 1:1 square
