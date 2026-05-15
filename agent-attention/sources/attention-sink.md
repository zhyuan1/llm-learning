# Attention Sink Phenomenon
Source: "Why do LLMs Attend to the First Token?" + related mechanistic analysis

## Core Mechanism
LLMs disproportionately assign high attention weights to the INITIAL token (e.g., <bos>),
even when that token has no semantic relevance to the current query.

## Why It Happens
- Softmax normalization forces attention scores to sum to 1
- When no token is highly relevant, model dumps excess attention onto stable initial token
- Acts like a "garbage collector" for unused attention budget
- Training dynamics: initial tokens serve as anchors that help stabilize gradient flow

## Connection to Lost-in-the-Middle
- Initial token absorbs ~46%+ of attention → middle tokens starved
- RoPE positional encoding favors recent tokens → end remains accessible
- Middle gets near-zero attention → effectively "lost"

Qwen's Gated Attention (NeurIPS 2025): reduced initial token attention from ~46.7% → 4.8%

---
# Mechanistic Explanation: Why Primacy and Recency Effects Exist

## Primacy (beginning bias)
- Causal masking in autoregressive models forces tokens to attend only to preceding tokens
- Early tokens have GLOBAL visibility — every subsequent token can attend to them
- Early tokens define the semantic "frame" for the sequence
- Their high initial attention weights propagate through layers

## Recency (end bias)
- Tokens near prediction point have shorter attention pathways → less signal decay
- RoPE positional encoding naturally decays attention weight with distance
- Models trained on next-token prediction develop strong recency instincts

## Why Middle Suffers
- Middle tokens have neither global visibility (primacy) nor proximity (recency)
- Attention sink steals budget from middle: initial tokens absorb ~46% of attention
- Softmax normalization: budget is zero-sum

## Architecture Specificity
- Decoder-only (GPT-style): most susceptible due to causal masking
- Encoder-decoder (T5/Flan): less susceptible due to bidirectional attention
- Mamba (SSM): similar effects but through different mechanism (exponential decay)
