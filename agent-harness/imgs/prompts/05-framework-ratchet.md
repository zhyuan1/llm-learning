---
type: framework
style: notion
palette: default
aspect: 1:1
language: zh
---

# Prompt

Notion-style minimalist hand-drawn framework diagram. White background, black ink, blue accent. Central metaphor: a
ratchet gear that only moves forward.

ZONES:

- CENTER: A large hand-drawn ratchet gear (pawl-and-ratchet mechanism sketch), labeled "棘轮原则" in the center of the
  gear. The pawl arrow points upward-right indicating one-way motion.
- LEFT of gear: A failure event box with dashed border
    - Title: "一次失败事件"
    - Example: "Agent 提交了注释掉测试的 PR"
    - Arrow pointing right toward the gear
- RIGHT of gear: Three stacked fix layers, each as a labeled box with arrow pointing outward (showing permanent fixes
  radiating outward)
    - Layer 1 (outermost, lightest): "Prompt 层" / "AGENTS.md 增加规则" / badge: "概率性"
    - Layer 2 (middle): "Hook 层" / "pre-commit 脚本阻断" / badge: "确定性 ✓"
    - Layer 3 (innermost, blue fill): "验证层" / "Reviewer 子Agent评审标准更新" / badge: "语义覆盖"
- BOTTOM: Two comparison callouts
    - Left: "Prompt 规则 → 模型有时会忘"
    - Right (highlighted): "Hook 检查 → 每次都执行，一次不漏"
- TOP: Small caption: "每次失败都工程化成永久修复"

LABELS: All Chinese. Key: 棘轮原则, Prompt层, Hook层, 验证层, 概率性, 确定性

COLORS: White background, black ink. Blue fill only for the innermost layer box and the right callout.

STYLE: Notion hand-drawn. The gear should be a recognizable ratchet sketch. Layers use concentric semi-circle layout or
stacked boxes with progressively bolder borders.

ASPECT: 1:1 square
