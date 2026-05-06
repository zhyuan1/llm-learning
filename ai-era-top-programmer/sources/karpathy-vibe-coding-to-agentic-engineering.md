---
title: "Karpathy: From Vibe Coding to Agentic Engineering"
author: Andrej Karpathy (via baoyu.io)
source: https://baoyu.io/blog/andrej-karpathy-from-vibe-coding-to-agentic-engineering
date: 2026-04-29 (AI Ascent 2026 interview, ~4 days before 2026-05-03)
---

# Karpathy's Talk on Vibe Coding and Agentic Engineering

## Overview

Comprehensive analysis of Andrej Karpathy's 30-minute interview at Sequoia Capital's AI Ascent (April 2026), where he
discusses the evolution from "Vibe Coding" to "Agentic Engineering" and the fundamental shift in how AI is transforming
software development.

---

## Key Turning Points

### The December 2025 Inflection Point

- Karpathy experienced a critical shift when AI-generated code became directly usable without modification
- He transitioned from manually correcting AI output to complete trust in the system
- This marks the beginning of true "Vibe Coding" - a development style where developers provide intent through natural
  language while AI handles code generation, modification, and debugging
- The shift represents moving from "ChatGPT-like Q&A" to coherent agentic workflows where models continuously plan,
  code, debug, and adapt

---

## Core Concepts

### Software 3.0 Framework

The evolution of software development across three eras:

**Software 1.0:** Traditional programming - humans write explicit code, computers execute rules

**Software 2.0:** Neural network era - humans design datasets and architectures; training produces model weights

**Software 3.0:** LLM era - LLMs become programmable computers where:

- Programming shifts from code files to context windows
- The "program" becomes: prompts, context, files, tool calls, and external environments
- Example: Instead of writing shell scripts for OpenCL installation, you provide installation instructions as text to an
  Agent, which reads the environment, executes steps, and debugs errors

**Key insight:** "Which text should you copy to your Agent? That's the new programming paradigm."

### The MenuGen Case Study

Karpathy's app that generates images of menu items to help diners visualize dishes:

- **Old approach:** OCR → extract dish names → generate images → reformat menu → deploy
- **Software 3.0 approach:** Send menu photo to Gemini with instruction to overlay generated images directly back onto
  the menu
- **Critical realization:** His entire MenuGen app was obsolete because it operated in the old paradigm
- **Business implication:** Many AI apps assume they're making things "faster," but Software 3.0 models can directly
  absorb entire application layers, making the intermediate app unnecessary

---

## LLM Capabilities: The "Jagged Intelligence" Problem

### Extreme Capability Asymmetry

LLMs demonstrate contradictory abilities:

- **Can:** Refactor 100,000 lines of code, find zero-day vulnerabilities
- **Cannot:** Correctly answer "Should I walk or drive 50 meters to wash my car?" (the model might say walk, ignoring
  that the car needs to reach the car wash)

### Why This Happens

Capability distribution depends on:

1. **Verifiability:** LLMs excel at tasks where outputs can be verified (math has answers, code has tests, security has
   exploits)
2. **RL Training Coverage:** Abilities spike in domains where labs invested in reinforcement learning data
3. **Data Distribution:** Model capabilities follow lab data decisions - e.g., GPT-3.5 to GPT-4's chess improvement came
   from adding chess data to pretraining, not "natural evolution"

### Practical Implication

Users must map the "ability landscape" - identifying which tasks fall within "capability peaks" versus "cliffs." You
cannot assume strength in one domain transfers to adjacent areas.

---

## Vibe Coding vs. Agentic Engineering

### Vibe Coding: Raising the Floor

- Democratizes software creation - non-programmers can build tools
- Lowers barriers to entry for side projects
- Represents "feeling your way through development" with AI assistance

### Agentic Engineering: Maintaining the Ceiling

- Ensures professional software maintains quality, safety, and accountability standards
- Treats Agents as "spiky entities" - powerful but error-prone and unstable
- Requires engineers to:
    - Design proper workflows with verification and rollback
    - Maintain system boundaries and constraints
    - Oversee Agent output quality
    - Ensure security and maintainability

**Key distinction:** Vibe Coding raises what everyone can do; Agentic Engineering preserves what professionals must
maintain.

---

## What Remains Human-Irreplaceable (For Now)

### The Specification Layer

Humans must own:

- **System design:** Understanding identity, payment, and data relationships (e.g., why binding payments to email
  addresses rather than persistent user IDs is dangerous)
- **Architectural decisions:** Knowing when to use internal IDs vs. external references
- **Risk identification:** Spotting design flaws that pass tests but fail in production

### Conceptual Understanding vs. API Details

- **Can be outsourced:** API names, syntax details (keepdims vs. keepdim, reshape vs. permute)
- **Cannot be outsourced:** Understanding tensor mechanics, view vs. storage relationships, memory efficiency
  implications

### Taste and Judgment

- Code generated by Agents often "works but is ugly" - bloated, copy-pasted, poorly abstracted
- Karpathy describes seeing Agent code as causing "heart attack feelings"
- Simplification tasks (like MicroGPT) feel like "pulling teeth" - suggesting they fall outside current RL training
  coverage
- These gaps may close if labs add aesthetic/quality objectives to RL training

---

## The "Animals vs. Ghosts" Framework

### What LLMs Actually Are

**Not:** Animal-like intelligence with emotions, motivations, or internal drives

**Actually:** Simulated entities shaped by:

- Large-scale pretraining on human documents
- Statistical pattern learning
- Reinforcement learning and preference data
- Tool calling and reward functions

### Practical Implications

- Yelling at an LLM has no effect (no fear response)
- Praising it doesn't activate motivation (no internal drive)
- Understanding the model as a statistical simulation helps predict where it will fail
- The framework prevents anthropomorphizing and enables better mental models for usage

---

## Entrepreneurial Opportunities

### The RL Coverage Gap

Most promising opportunities exist in domains that are:

1. **Verifiable** - outputs can be evaluated
2. **Not yet covered** by major lab RL training
3. **Constructible** - you can build reward environments

### Karpathy's Hint (Without Revealing)

He deliberately avoided naming specific sectors but emphasized that almost everything could eventually become verifiable
through mechanisms like "LLM judges" (model review panels) for subjective tasks like writing and design.

---

## Agent-First Infrastructure

### Current State

Today's tools are human-centric:

- Documentation tells you: "Go to URL → click setting → copy key → configure DNS"
- Deployment requires manual navigation through multiple interfaces

### Future State

Infrastructure should be Agent-native:

- Provide text that Agents can directly consume
- Expose state and actions through APIs Agents understand
- Enable Agents to complete full workflows (code → deploy → configure → monitor) without human menu navigation

### Vision

- One-sentence deployment: "Build MenuGen" → complete, deployed application
- Agent representations for individuals/organizations handling coordination
- Seamless integration across services without manual configuration

---

## The Learning Paradox

### The Core Tension

> "You can outsource your thinking, but you cannot outsource your understanding."

### What Changes

As intelligence becomes cheaper:

- **Less important:** Mechanical memorization, low-level execution details
- **More important:** System understanding, problem definition, quality judgment, causal reasoning, domain intuition

### The Bottleneck Shift

Karpathy observes he's becoming the bottleneck:

- Must understand what's being built and why it matters
- Must guide Agents effectively
- Must catch errors in identity binding, system structure, and code quality
- Cannot delegate the judgment of "is this worth doing?"

---

## Critical Tensions and Open Questions

### Three Unresolved Tensions

1. **The Ugliness Paradox:** Code works but is often poorly designed, yet Karpathy uses it anyway - suggesting real Vibe
   Coding tolerates quality compromises

2. **The Hidden Opportunity:** Karpathy hints at undervalued RL opportunities but won't name them publicly - suggesting
   a narrow window remains open

3. **The Replaceability Question:** Human judgment seems irreplaceable only because labs haven't optimized for it yet -
   raising questions about whether this advantage is permanent

### Three Signals to Watch (Next 6-12 Months)

1. **RL Data Expansion:** Which domains beyond coding/math receive RL investment? Capability will spike there.

2. **Agent-First Infrastructure:** Will deployment/auth/payments infrastructure converge on Agent-native patterns? If
   not, automation remains incomplete.

3. **Quality-Focused RL:** Will next-generation models include aesthetic and code-quality objectives in RL training?

---

## Bottom Line

Karpathy's core argument: **The next phase of AI isn't about faster coding—it's about maintaining software quality while
Agents accelerate execution.** The real scarcity shifts from execution capability to judgment about what's worth
building and how to architect systems safely. Understanding remains the irreplaceable human contribution, at least until
training methods evolve further.
