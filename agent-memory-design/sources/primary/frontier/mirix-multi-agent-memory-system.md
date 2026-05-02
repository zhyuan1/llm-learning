---
title: "MIRIX: Multi-Agent Memory System for LLM-Based Agents"
url: "https://arxiv.org/abs/2507.07957"
retrieved: "2026-05-02"
source_type: "primary"
---

- URL: https://arxiv.org/abs/2507.07957
- 类型: 论文
- 记录日期: 2026-05-02
- PDF: https://arxiv.org/pdf/2507.07957.pdf

# MIRIX: Multi-Agent Memory System for LLM-Based Agents

Authors: Yu Wang, Xi Chen

Published: 2025-07-10
Updated: 2025-07-10

## Abstract

Although memory capabilities of AI agents are gaining increasing attention, existing solutions remain fundamentally
limited. Most rely on flat, narrowly scoped memory components, constraining their ability to personalize, abstract, and
reliably recall user-specific information over time. To this end, we introduce MIRIX, a modular, multi-agent memory
system that redefines the future of AI memory by solving the field's most critical challenge: enabling language models
to truly remember. Unlike prior approaches, MIRIX transcends text to embrace rich visual and multimodal experiences,
making memory genuinely useful in real-world scenarios. MIRIX consists of six distinct, carefully structured memory
types: Core, Episodic, Semantic, Procedural, Resource Memory, and Knowledge Vault, coupled with a multi-agent framework
that dynamically controls and coordinates updates and retrieval. This design enables agents to persist, reason over, and
accurately retrieve diverse, long-term user data at scale. We validate MIRIX in two demanding settings. First, on
ScreenshotVQA, a challenging multimodal benchmark comprising nearly 20,000 high-resolution computer screenshots per
sequence, requiring deep contextual understanding and where no existing memory systems can be applied, MIRIX achieves
35% higher accuracy than the RAG baseline while reducing storage requirements by 99.9%. Second, on LOCOMO, a long-form
conversation benchmark with single-modal textual input, MIRIX attains state-of-the-art performance of 85.4%, far
surpassing existing baselines. These results show that MIRIX sets a new performance standard for memory-augmented LLM
agents. To allow users to experience our memory system, we provide a packaged application powered by MIRIX. It monitors
the screen in real time, builds a personalized memory base, and offers intuitive visualization and secure local storage
to ensure privacy.
