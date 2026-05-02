---
type: framework
style: minimal-flat
palette: default-soft
language: zh
aspect: 16:9
output: ../04-framework-hermes-memory.png
---

Create a minimal-flat architecture diagram explaining Hermes Agent memory design in Chinese.

TITLE: “Hermes：小而稳的常驻记忆层”

LAYOUT:

- Left side: two small file cards:
    1. “MEMORY.md：环境 / 项目 / 经验” with “2200 chars” label
    2. “USER.md：用户画像 / 偏好” with “1375 chars” label
- These flow into a central large box:
  “Frozen Snapshot（会话开始时注入）”
- From the central box, arrow to:
  “System Prompt / 当前会话”
- Below, a separate side channel:
  “SQLite 会话库 + FTS5” → “session_search” → “LLM 摘要” → “按需进入上下文”
- Right bottom: “Context Compression + Prompt Caching” shown as a stabilizer around system prompt.

KEY ANNOTATIONS:

- “常驻记忆：小、硬、稳定”
- “历史细节：旁路检索”
- “中途写入落盘，下次 session 生效”

STYLE:
Minimal flat vector architecture, clean and precise, soft blue-gray palette, orange accent for Frozen Snapshot, green
accent for session_search.

TYPOGRAPHY:
Chinese labels clear and legible.

ASPECT:
16:9 widescreen for WeChat.
