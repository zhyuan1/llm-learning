---
title: "ReAct: Synergizing Reasoning and Acting in Language Models"
authors: Shunyu Yao et al.
source: arXiv:2210.03629 / ICLR 2023
date: 2023
url: https://arxiv.org/abs/2210.03629
---

## Abstract

We explore the use of LLMs to generate both reasoning traces and task-specific actions in an interleaved manner,
allowing for greater synergy: reasoning traces help the model induce, track, and update action plans as well as handle
exceptions, while actions allow it to interface with external sources to gather additional information.

## Core Contribution

The **Thought → Action → Observation** execution loop: the foundational pattern underpinning most modern agent
harnesses.

## Key Results

- HotpotQA: ReAct **overcomes hallucination and error propagation** prevalent in chain-of-thought reasoning by
  interacting with a simple Wikipedia API
- ALFWorld: outperforms imitation and RL methods by **absolute success rate of 34%**
- WebShop: outperforms by **10% absolute success rate**
- Only 1-2 in-context examples needed

## Why This Matters for Harness Design

Before ReAct, agents either reasoned (chain-of-thought) OR acted (action plan generation). ReAct showed these must be
interleaved. The **loop structure** — not the model — is the control variable for reducing hallucinations.

The execution loop is the skeleton of the harness. Every other component (tools, memory, observation format) plugs into
this loop.
