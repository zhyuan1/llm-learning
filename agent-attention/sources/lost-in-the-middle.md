# Lost in the Middle: How Language Models Use Long Contexts
Authors: Nelson F. Liu et al. (Stanford)
Published: 2023, TACL
URL: https://arxiv.org/abs/2307.03172

## Core Finding
U-shaped performance curve: models perform best when relevant info is at beginning or end; significantly degrades when info is in the MIDDLE.

## Tasks Evaluated
1. Multi-document QA — accuracy drops when answer doc is placed mid-context
2. Key-value retrieval — fails when keys are positioned centrally

## Architecture Insights
- Decoder-only models (GPT-style) more susceptible to positional bias
- Encoder-decoder models (Flan-T5) more robust due to bidirectional attention
- Longer contexts EXACERBATE the problem

## Practical Implication
Even models explicitly designed for long contexts (Claude-1.3, GPT-3.5-turbo-16k) show this effect.
