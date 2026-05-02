---
title: "Memory OS of AI Agent"
url: "https://arxiv.org/abs/2506.06326"
retrieved: "2026-05-02"
source_type: "primary"
---

- URL: https://arxiv.org/abs/2506.06326
- 类型: 论文
- 记录日期: 2026-05-02
- PDF: https://arxiv.org/pdf/2506.06326.pdf

# Memory OS of AI Agent

Authors: Jiazheng Kang, Mingming Ji, Zhe Zhao, Ting Bai

Published: 2025-05-30
Updated: 2025-05-30

## Abstract

Large Language Models (LLMs) face a crucial challenge from fixed context windows and inadequate memory management,
leading to a severe shortage of long-term memory capabilities and limited personalization in the interactive experience
with AI agents. To overcome this challenge, we innovatively propose a Memory Operating System, i.e., MemoryOS, to
achieve comprehensive and efficient memory management for AI agents. Inspired by the memory management principles in
operating systems, MemoryOS designs a hierarchical storage architecture and consists of four key modules: Memory
Storage, Updating, Retrieval, and Generation. Specifically, the architecture comprises three levels of storage units:
short-term memory, mid-term memory, and long-term personal memory. Key operations within MemoryOS include dynamic
updates between storage units: short-term to mid-term updates follow a dialogue-chain-based FIFO principle, while
mid-term to long-term updates use a segmented page organization strategy. Our pioneering MemoryOS enables hierarchical
memory integration and dynamic updating. Extensive experiments on the LoCoMo benchmark show an average improvement of
49.11% on F1 and 46.18% on BLEU-1 over the baselines on GPT-4o-mini, showing contextual coherence and personalized
memory retention in long conversations. The implementation code is open-sourced at https://github.com/BAI-LAB/MemoryOS.
