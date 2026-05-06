---
illustration: 01
type: infographic
style: blueprint
palette: default
language: zh
aspect: 16:9
article: agent-long-task/draft.md
section: 一、两组数据
---

# Prompt: 01-infographic-benchmark-data

Blueprint-style technical infographic on dark blue grid background with white/cyan technical lines and schematic
aesthetic.

## LAYOUT

Two-column data comparison layout with a shared bottom summary bar.

## ZONES

### HEADER

- Title text: "AI Agent 长任务基准测试"
- Subtitle: "两组独立研究，相同结论"
- Style: top-left aligned, white monospace bold title, cyan thin subtitle

### LEFT COLUMN — TravelPlanner (ICML 2024)

- Section label: "TravelPlanner · GPT-4"
- Large central metric: "0.6%" in oversized white bold font (blueprint highlight color: cyan)
- Below: small label "最终通过率（1225个任务）"
- Comparison bar chart:
    - Bar 1: "预期感知" → ~60% height, dashed outline, dim color
    - Bar 2: "实际通过率" → 0.6% height, solid cyan fill
- Small note: "逐条核查所有约束是否满足"
- Corner icon: small airplane/route schematic icon (blueprint line-art style)

### RIGHT COLUMN — UltraHorizon (2025)

- Section label: "UltraHorizon · 多模型测试"
- Two metric boxes side by side:
    - Box A: "人类" → score "26.52" in white
    - Box B: "Agent" → score "14.33" in dim cyan
- Arrow between them pointing from Agent → Human with label "差距 ~46%"
- Below: scale note: "最大配置: 200,000 tokens · 400次工具调用"
- Corner icon: small horizontal bar chart schematic (blueprint line-art)

### BOTTOM SUMMARY BAR

- Full-width horizontal bar, slightly elevated background
- Central stat: "56%" in large cyan bold
- Label: "的失败归因于上下文管理失败，而非推理能力不足"
- Small icon: document/context stack schematic on left

## STYLE RULES

- Background: dark navy blue with subtle blueprint grid lines — no color codes or hex values should appear anywhere in
  the image
- Primary color: white for key metrics and labels
- Accent color: bright cyan for highlights and bars
- Secondary: dim cyan for comparison/lesser values
- Font: monospace/technical feel throughout
- All borders: thin cyan or white lines, blueprint schematic style
- No corner labels, watermarks, color codes, or hex strings anywhere in the image
- No photorealistic elements, pure schematic/technical illustration
- Aspect ratio: 16:9
