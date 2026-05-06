---
title: "Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity"
author: Joel Becker, Nate Rush, Beth Barnes, David Rein
source: https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/
arxiv: https://arxiv.org/abs/2507.09089
date: 2025-07-10
---

# METR Study: AI Impact on Open-Source Developer Productivity

## Overview

METR conducted a randomized controlled trial (RCT) to measure how early-2025 AI tools affect the productivity of
experienced open-source developers. The surprising finding: **developers using AI tools took 19% longer to complete
tasks than without AI assistance**.

---

## Methodology

**Study Design:**

- **Participants:** 16 experienced developers from large open-source repositories (averaging 22,000+ stars and 1M+ lines
  of code)
- **Tasks:** 246 real issues (bug fixes, features, refactors) from developers' own repositories
- **Average task duration:** ~2 hours per issue
- **Randomization:** Each issue randomly assigned to either allow or disallow AI use
- **AI Tools Used:** Primarily Cursor Pro with Claude 3.5/3.7 Sonnet (frontier models at study time)
- **Compensation:** $150/hour for developer participation
- **Documentation:** Screen recordings and self-reported implementation times

---

## Core Findings

**Primary Result:**

- Developers with AI access took **19% longer** to complete issues
- This contradicts developer expectations: they predicted AI would speed them up by 24%
- Even after experiencing the slowdown, developers still believed AI had sped them up by 20%

**Quality Control:**

- Developers used frontier models as intended
- Complied with treatment assignments
- Didn't differentially drop issues based on difficulty
- Submitted similar quality PRs with and without AI
- Slowdown persisted across different outcome measures and analytical approaches

---

## Factor Analysis

The study investigated 20 potential factors explaining the slowdown and found evidence that **5 likely contribute** to
the result (specific factors detailed in the paper's factor analysis table).

---

## What the Study Does NOT Claim

The researchers explicitly clarified limitations:

- Does not prove AI doesn't speed up most developers generally
- Does not claim their developers/repositories represent majority of software development
- Does not address AI impact in non-software domains
- Does not predict future AI capabilities
- Does not rule out more effective ways to use existing AI systems

---

## Reconciling Contradictory Evidence

The study acknowledges three different sources of evidence give conflicting signals:

| Source                               | Finding                                                     |
|--------------------------------------|-------------------------------------------------------------|
| **This RCT**                         | Models slow down humans on realistic 20min-4hr coding tasks |
| **Benchmarks** (SWE-Bench, RE-Bench) | Models succeed at very difficult benchmark tasks            |
| **Anecdotes/Adoption**               | Many users report AI is helpful for substantial tasks       |

**Three hypotheses for reconciliation:**

1. **RCT underestimates:** Methodological issues or setting differences explain the gap
2. **Benchmarks/anecdotes overestimate:** Benchmark tasks are too narrow; anecdotal reports are inaccurate
3. **Complementary evidence:** All three are correct but measure different task distributions

---

## Key Insights from Discussion

- RCT results may be less relevant in settings where developers can sample hundreds/thousands of model trajectories
- Learning effects for tools like Cursor may require hundreds of hours of usage (developers averaged ~50 hours)
- AI capabilities may be lower in high-quality-standard settings with implicit requirements (documentation, testing,
  linting)
- Benchmarks may overestimate by only measuring well-scoped, algorithmically scorable tasks
- Anecdotal reports of speedup can be significantly inaccurate

---

## Future Direction

METR plans to:

- Run similar studies to track speedup/slowdown trends over time
- Use this methodology as a complement to benchmarks for more realistic evaluation
- Monitor whether AI systems eventually achieve substantial developer speedup (which could signal rapid AI R&D
  acceleration)
- Explore similar experiments in other settings

---

## FAQ Highlights

**Why were developers slowed down if they could choose not to use AI?**

- Developers misestimated AI's impact (thought it helped when it didn't)
- May use AI for reasons beyond productivity (enjoyment, skill-building for future systems)

**Statistical validity concerns:**

- Study used clustered standard errors accounting for 16 developers
- 246 total issues provided sufficient statistical power
- Acknowledged potential sampling biases in developer recruitment

**Generalizability:**

- Developers were experienced with Cursor/LLMs (dozens to hundreds of hours prior experience)
- Results specific to experienced developers on familiar codebases; may not apply to beginners or unfamiliar codebases
