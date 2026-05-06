---
title: "METR Developer Productivity Experiment Design Update"
author: Joel Becker, Nate Rush, Tom Cunningham, David Rein, Khalid Mahamud
source: https://metr.org/blog/2026-02-24-uplift-update/
date: 2026-02-24
---

# METR Developer Productivity Experiment Design Update — February 2026

## Overview

METR announcing significant changes to their experimental design due to methodological challenges discovered in their
latest study. This is a follow-up to the July 2025 RCT.

## Key Findings & Numbers

### Early 2025 Study Results (recap)

- **Finding**: AI tools caused a **20% slowdown** in task completion among experienced open-source developers
- **Study Period**: February to June 2025
- **Confidence Interval**: +2% to +39% longer completion time

### Late 2025 Study Results (August 2025 onwards)

The new experiment revealed more complex results with notable selection bias issues:

- **Original Developers Subset**: Estimated speedup of **-18%** (confidence interval: -38% to +9%)
- **Newly-Recruited Developers**: Estimated speedup of **-4%** (confidence interval: -15% to +9%)
- **Study Size**: 57 developers total (10 from original study + 47 new), across 143 repositories, 800+ tasks
- **Compensation**: Reduced from $150/hour to $50/hour

## Critical Issues Identified

### Primary Problems

1. **Selection Bias - Recruitment**: Increased share of developers unwilling to work without AI, even at $50/hour pay.
   The study systematically excludes developers with highest AI expectations.

2. **Selection Bias - Task Selection**: 30-50% of developers reported avoiding submission of tasks they didn't want to
   complete without AI, creating systematic gaps in data on high-uplift tasks.

3. **Measurement Reliability**: Time tracking became unreliable when developers used multiple AI agents concurrently, as
   they would work on unrelated tasks while waiting for agents to complete work.

### Secondary Issues

- Task types differ between AI-allowed and AI-disallowed conditions
- Quality of work varies between conditions (code quality, documentation, tests)
- Some developers failed to complete tasks assigned to AI-disallowed condition
- Pay rate reduction likely contributed to selection effects

## Key Insights from Developer Quotes

The article includes three representative quotes from study participants illustrating:

- Reluctance to participate without AI access
- Conscious avoidance of tasks where AI would provide significant advantage
- Difficulty adapting to non-AI workflows after becoming accustomed to AI assistance

## Future Research Directions

METR plans to pursue six alternative or complementary approaches:

1. **More intensive experiments** with higher compliance and potentially higher pay rates
2. **Observational data** from existing sources (e.g., GitHub commit statistics showing ~4% of commits authored by
   Claude Code)
3. **Questionnaires** and time-use studies despite self-reporting limitations
4. **Fixed-task experiments** with predetermined tasks rather than developer-selected ones
5. **Evaluations** measuring autonomous agent task completion capabilities
6. **Developer-level randomization** instead of task-level (though with acknowledged tradeoffs)

## Data Availability

Both datasets are publicly available on GitHub:

- Early 2025 study: https://github.com/METR/Measuring-Early-2025-AI-on-Exp-OSS-Devs
- Late 2025 study: https://github.com/METR/Measuring-Late-2025-AI-on-OSS-Devs
