---
title: "SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering"
authors: John Yang, Carlos E. Jimenez, Alexander Wettig, Kilian Lieret, Shunyu Yao, Karthik Narasimhan, Ofir Press
source: arXiv:2405.15793
date: 2024-05-06
url: https://arxiv.org/abs/2405.15793
---

## Abstract

Language model (LM) agents are increasingly being used to automate complicated tasks in digital environments. Just as
humans benefit from powerful software applications, such as integrated development environments, for complex tasks like
software engineering, we posit that LM agents represent a new category of end users with their own needs and abilities,
and would benefit from specially-built interfaces to the software they use. We investigate how interface design affects
the performance of language model agents. As a result of this exploration, we introduce SWE-agent: a system that
facilitates LM agents to autonomously use computers to solve software engineering tasks. SWE-agent's custom
agent-computer interface (ACI) significantly enhances an agent's ability to create and edit code files, navigate entire
repositories, and execute tests and other programs. We evaluate SWE-agent on SWE-bench and HumanEvalFix, achieving
state-of-the-art performance on both with a pass@1 rate of 12.5% and 87.7%, respectively, far exceeding the previous
state-of-the-art achieved with non-interactive LMs.

## Key Data Points

- SWE-bench pass@1: **12.5%** (vs RAG baseline ~3.8% with same GPT-4)
- HumanEvalFix pass@1: **87.7%**
- Non-interactive LM baselines: much lower (confirming harness is the differentiator)

## ACI Architecture (from swe-agent.com/background/aci)

1. **Linter integration**: Syntax validator prevents faulty code edits from executing
2. **Specialized file viewer**: ~100 lines per turn with scrolling/search (NOT raw cat)
3. **Directory search command**: Shows only filenames with matches (not verbose context)
4. **Empty output handling**: Explicit confirmation messages when commands return nothing

## Design Principle

> "good ACI design leads to much better results when using agents"
> "a baseline agent without a well-tuned ACI does much worse than SWE-agent"

## From Anthropic SWE-bench blog

- Two primary tools: **Bash** (execute commands) + **Edit** (string replacement)
- Requiring **absolute filepaths** eliminated path confusion errors
- Design philosophy: minimal scaffolding, maximum model autonomy
- Claude 3.5 Sonnet (updated) + this harness = **49%** on SWE-bench Verified
- Previous SOTA: 45%
- Claude 3.5 Sonnet (older): 33%
- Claude 3 Opus: 22%

> "the performance of an agent on SWE-bench can vary significantly based on this scaffolding, even when using the same
> underlying AI model"
