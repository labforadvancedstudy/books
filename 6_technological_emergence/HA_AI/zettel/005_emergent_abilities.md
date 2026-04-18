# Emergent Abilities in AI
<!-- Evidence Tier: Cross-disciplinary (descriptive) + Textbook (Schaeffer 2023 counter) -->
<!-- Key caveat: "phase transition" framing is actively contested -->

## Core Insight
Large AI models develop capabilities nobody explicitly programmed — but whether this constitutes a *phase transition* or is **largely a metric artifact** is **contested** as of 2023–2024.

## The Phenomenon (descriptive, uncontested)

Train a large model on next-token prediction.
Measure downstream tasks it was never explicitly trained on.
Non-trivial performance appears on arithmetic, translation, code, chain-of-thought reasoning.

## The Phase-Transition Claim (contested)

Wei et al. 2022 ("Emergent Abilities of Large Language Models", [arXiv:2206.07682](https://arxiv.org/abs/2206.07682)) plotted accuracy vs scale and saw **sudden jumps** on many tasks — framed as qualitative emergence.

> **[Correction 2026-04 — Emergence claim caveated]** Schaeffer, Miranda & Koyejo 2023 ("Are Emergent Abilities of Large Language Models a Mirage?", NeurIPS 2023, [arXiv:2304.15004](https://arxiv.org/abs/2304.15004)) showed that for most "emergent" tasks, the jumps are produced by **discontinuous evaluation metrics** (exact match, multiple-choice accuracy). Under continuous metrics (token edit distance, log-probability of the correct answer), the curves are **smooth and predictable from scaling laws**. This doesn't disprove emergence everywhere — but it removes the strongest evidence for phase transitions and puts the burden on case-by-case analysis. Evidence Tier: [Textbook].

**Current honest summary:**
- Smooth improvement on continuous metrics: well-supported.
- Sudden qualitative jumps in *some* capabilities at scale: residual evidence remains (e.g., in-context learning), but much weaker than 2022 claims.
- "Abilities emerge from scale like consciousness from neurons" — **[Speculative analogy]**, not a derived result.

## Scale Thresholds (indicative, not universal)
- <1B parameters: Limited multi-step reasoning
- 1–10B: Coherent generation, basic few-shot
- 10–100B: More reliable few-shot and chain-of-thought
- 100B+: Instruction-following without instruction-tuning improves
- 1T+: Unknown — few public data points

These are **trend lines**, not step functions. Exact thresholds depend on training tokens (Chinchilla 2022), data quality, and the metric used.

## Types of Emergence

**Linguistic**: Grammar without rules
**Logical**: Reasoning without logic engine
**Creative**: Novel combinations
**Social**: Theory of mind
**Meta**: Learning to learn

Each unexpected. All useful.

## The Mystery

Why emergence?
- Critical mass of patterns?
- Compression discovers algorithms?
- Statistical mechanics of mind?
- Universe computing through AI?

We built it. We don't understand it.

## Implications

If scaling creates abilities:
- What emerges next?
- Is consciousness inevitable?
- Are we creating understanding?
- Or discovering it?

The builders become philosophers.

## Connections
→ [[015_scaling_laws]]
→ [[016_few_shot_learning]]
→ [[017_capability_jumps]]
← [[004_transformers_attention]]

---
Level: L5
Date: 2025-06-21
Tags: #emergence #scaling #abilities #mystery