---
illustration: 02
type: framework
style: blueprint
palette: default
language: zh
aspect: 16:9
article: agent-long-task/draft.md
section: 三、上下文锁定
---

# Prompt: 02-framework-context-locking

Blueprint-style technical framework diagram explaining the "in-context locking" phenomenon in LLMs. Dark navy background
with schematic line art.

## LAYOUT

Two-part mechanism diagram (top + bottom) feeding into a central "locking effect" box, with a right-side annotation
panel.

## ZONES

### HEADER

- Title: "上下文锁定机制"
- Subtitle: "In-Context Locking — 两个底层原因"
- Style: white monospace bold, top-centered

### MECHANISM 1 — TOP HALF: 注意力稀释 (Attention Dilution)

- Label: "机制一：注意力权重稀释"
- Visual: horizontal token sequence diagram
    - Left side: 5-6 early token blocks (solid white outline, labeled "早期假设 Token")
    - Right side: growing sequence of later tokens (more blocks, smaller labels)
    - Arrow annotations above: "注意力权重" with downward gradient arrow (thick at start, thin toward end)
    - Key label: "上下文越长 → 早期Token的有效注意力越薄"
    - NOT disappearing: small note "早期内容并未消失，只是影响力变薄"
- Style: schematic register/memory diagram aesthetic, cyan fill for early tokens, dim for later tokens

### MECHANISM 2 — BOTTOM HALF: 路径依赖 (Path Dependency)

- Label: "机制二：路径依赖"
- Visual: chain of dependency boxes
    - Box 1 (left): "早期错误假设" — solid red-orange outline, warning icon
    - Arrow →
    - Box 2: "写入上下文" — gets solidified/frozen icon
    - Arrow →
    - Box 3: "约束后续每步生成" — chain link icon
    - Arrow →
    - Box 4 (right): "新信息被纳入旧框架" — lock icon, dimmed input signal
- Bottom note: "条件概率链：下一个token以所有已生成token为条件"

### CENTER CONVERGENCE: 锁定效应

- Diamond or hexagon shape in the middle connecting both mechanisms
- Label inside: "上下文锁定"
- Subtitle: "早期内容：难以被重新审视 + 难以被推翻"
- Double arrow from both mechanisms converging here
- Glow effect: cyan outline, slightly brighter than surrounding

### RIGHT PANEL: 临界点标注

- Vertical bar chart or gauge:
    - Y-axis: "上下文占用 (%)"
    - Marked zones:
        - 0-40%: "正常区域" (green-ish dim)
        - 40%: dashed red line with label "性能断崖"
        - 40-100%: "锁定效应显著增强" (orange-red dim)
- Source note: "UltraHorizon 分析结论"

## STYLE RULES

- Background: dark navy (#0a1628) with blueprint grid
- Mechanism 1: cyan (#00d4ff) schematic lines
- Mechanism 2: orange (#ff8c42) for error/warning elements, white for neutral
- Center box: bright cyan with subtle glow
- Right panel: standard blueprint schematic
- All text: white or light cyan, monospace feel
- Connecting lines: technical arrow style with clear directionality
- Aspect ratio: 16:9
