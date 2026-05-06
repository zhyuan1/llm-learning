---
title: "Developer Taste: Separating Good Code from AI Slop"
author: Fran Soto
source: https://strategizeyourcareer.com/p/developer-taste-ai-slop
date: 2026-04-18
---

# Developer Taste: Separating Good Code from AI Slop

## Summary

This article explores the concept of "developer taste" — the engineering judgment to recognize what good code looks like
before writing it — and argues this skill is now more critical than ever in the AI era. The piece distinguishes between
engineers who use AI to ship faster versus those who use it to ship better, emphasizing that AI slop is fundamentally a
taste problem, not an AI problem.

---

## Core Definition: What is Developer Taste?

**Developer taste** is defined as: *the judgment to know what the right thing is, and the discipline to pursue it,
before you write a single line of code.*

Key distinctions:

- **Taste vs. Skill**: Taste is the judgment about whether something is worth building; skill is the ability to build
  what's described. Many engineers have skill without taste.
- **Taste vs. Speed**: Speed measures how fast you ship; taste measures how often what you ship was worth shipping in
  the first place.
- **Taste vs. Aesthetics**: It's not about code style, formatting preferences, or nitpicking — it's about architectural
  and design judgment.

---

## The AI Slop Problem

### What is AI Slop?

AI slop is code that:

- Compiles and passes tests but makes no sense upon careful reading
- Works today but quietly creates problems for the next six months
- Solves problems nobody asked about
- Duplicates logic already existing elsewhere
- Handles only the happy path with no error handling

**Critical insight**: Slop isn't broken code (which gets caught). It's code that works today and becomes technical debt
tomorrow.

### Why It's Getting Worse

- AI raised the floor on how fast code can be produced
- AI did nothing for how carefully code must be thought through
- Companies increased delivery pressure thanks to AI capabilities
- Most engineers kept their old thinking patterns but now use prompts instead of typing
- The thinking got forgotten

---

## Five Common Taste Mistakes with AI

1. **Treating AI output as final** — Accepting whatever the model produces without reviewing whether it's what you would
   have written. "Vibe coding" without critical evaluation.

2. **Copying from secondary sources instead of primary ones** — Pattern-matching against similar code in the repo rather
   than consulting the actual specification, documentation, RFC, or design review.

3. **Skipping problem decomposition** — Asking AI to solve all three parts of a task at once instead of breaking it into
   pieces you can reason about separately.

4. **Shipping the happy path only** — Code that works when everything goes right but lacks error handling, edge case
   management, and comprehensive testing.

5. **Making code work without making it right** — The "50-50 rule": getting code to work is half the job; making it
   clean, small, reviewable, and understandable in six months is the other half.

---

## Practical Examples of Taste in Action

**Example 1 — Architecture Awareness**: An engineer caught AI adding types to the wrong file because it didn't
understand the service had two separate API surfaces that should never share definitions. The code was technically
correct but architecturally wrong.

**Example 2 — Thoughtful Prompting**: When building a feature, an engineer paused before pasting AI code, wrote down
what the feature actually needed end-to-end, reviewed existing integration tests, and only then returned to AI with a
much more detailed prompt. The result was a change four times smaller and five times better than the initial AI
suggestion.

---

## How to Develop Developer Taste

### Six Practical Practices

1. **Work outside-in** — Write integration tests describing external behavior before implementation. This forces you to
   define "done" before starting and gives you a truth function to test AI output against.

2. **Keep commits small and single-purpose** — One commit, one responsibility. This forces you to have opinions about
   each change, which is where taste lives.

3. **Review your own code as a reader** — Before assigning a PR, read it as if someone else wrote it. Ask what questions
   you'd ask a junior engineer. This is the best way to spot AI slop in your own work.

4. **Go to primary sources** — Consult documentation, standards, papers, and ADRs rather than relying on the model's
   training knowledge.

5. **Define the skeleton yourself, let AI fill it in** — Decide types, function signatures, module boundaries, and
   names. Let AI implement the bodies. This inverts the default workflow.

6. **Review more code than you write** — Reading others' code, especially code you think is "bad," develops your nose
   for what's wrong and calibrates you against the full distribution.

---

## Career Implications: Two Diverging Paths

### Engineers Without Taste → Executors

- Will ship lots of code and look busy
- Will hit sprint metrics
- Treated as fungible resources
- Market rate for typing is dropping fast

### Engineers With Taste → Orchestrators

- Frame problems and design shapes
- Review output with clear intent and firm opinions
- Their leverage multiplies as tools improve
- Market rate for judgment is climbing

**The prediction**: The next five years will be rough for the first group and spectacular for the second. The separator
is not talent, tenure, or company — it's whether they developed their taste.

---

## Key Takeaways

- **Taste is learnable**, not innate — it's a practice built through deliberate, consistent effort
- **AI amplifies existing judgment** — it makes good engineers better and enables poor engineers to ship more mediocre
  code faster
- **The thinking matters more than the typing** — AI changed the tool but not the underlying skill
- **Context and history are irreplaceable** — AI can't see why architectural boundaries exist or why past decisions were
  made
- **Taste compounds over time** — small deliberate practices build into significant competitive advantage
