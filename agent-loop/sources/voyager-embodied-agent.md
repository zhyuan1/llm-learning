# Voyager: An Open-Ended Embodied Agent with Large Language Models

来源：https://arxiv.org/abs/2305.16291
获取时间：2026-05-21

---

## Three-Component Architecture

1. **Automatic Curriculum**: "an automatic curriculum that maximizes exploration" — the agent generates its own goals based on current state and capabilities, without human-designed task lists.

2. **Skill Library**: "an ever-growing skill library of executable code for storing and retrieving complex behaviors" — successful code gets stored and indexed for future reuse.

3. **Iterative Prompting Mechanism**: "a new iterative prompting mechanism that incorporates environment feedback, execution errors, and self-verification for program improvement" — code is refined through multiple rounds.

## Lifelong Learning Without Fine-Tuning

"Voyager interacts with GPT-4 via blackbox queries, which bypasses the need for model parameter fine-tuning."

Learning happens through:
- Accumulating verified skills in the library (not in model weights)
- Each skill = executable JavaScript code that was tested and verified
- Skills compose: complex behaviors built from simpler verified skills

## Perception-Action-Feedback Loop in Minecraft

```
Perception: Game state (inventory, nearby blocks, health, position, time)
     ↓
Curriculum: "What should I do next?" → Generate goal
     ↓
Action: Generate JavaScript code to achieve goal
     ↓
Execute: Run code in Minecraft
     ↓
Feedback: 
  - Execution error? → Fix code (iterative prompting)
  - Self-verification failed? → Retry with different approach
  - Success? → Store skill in library
     ↓
Back to Perception (new state after action)
```

## Key Design Decisions

- **Code as action** (not JSON tool calls): more composable, verifiable, storable
- **Environment feedback > self-assessment**: compiler errors and game state changes are ground truth
- **Skill library = long-term memory**: indexed by description, retrieved by similarity
- **No reward signal needed**: self-verification replaces reward functions

## Results

- Obtains 3.3× more unique items than prior methods
- Traverses 2.3× longer distances
- Unlocks key tech tree milestones (diamond pickaxe) that no other method achieves
- Skills transfer to new worlds without retraining

## Significance for Agent Architecture

Voyager demonstrates the complete four-layer loop:
- Perception: game state observation
- Decision: curriculum + skill retrieval
- Action: code generation and execution
- Feedback: execution errors + self-verification → skill library update

The skill library is the key innovation: it turns episodic feedback into reusable procedural knowledge.
