# Scaling Laws
<!-- Evidence Tier: Textbook (Kaplan 2020, Chinchilla 2022, Schaeffer 2023) -->
<!-- Speculative sub-claim flagged inline: "intelligence has equations" / "inevitable at sufficient scale" -->

## Core Insight
Within a regime (dense transformer, pretraining loss, fixed data mixture), **loss** falls as a roughly additive power law in parameters (N) and training tokens (D). Not "capability" — **loss**. Not across any architecture — within one.

```
Loss(N, D) ≈ L∞ + A · N^(-α) + B · D^(-β)
```

**Key papers:**
- Kaplan et al. 2020 ([arXiv:2001.08361](https://arxiv.org/abs/2001.08361)) — first formulation; implied ~1.7 tokens/parameter as compute-optimal.
- Hoffmann et al. 2022 "Chinchilla" ([arXiv:2203.15556](https://arxiv.org/abs/2203.15556)) — **corrected to ~20 tokens/parameter**. Most earlier big models were **undertrained ~10×**.
- Schaeffer, Miranda & Koyejo 2023 ([arXiv:2304.15004](https://arxiv.org/abs/2304.15004)) — "emergent ability" jumps often disappear under continuous metrics. Emergence is partly a plotting artifact.

> **[Correction 2026-04 — Philosophical overreach trimmed]** Previous text said "Performance improves predictably … Intelligence has equations" and "consciousness inevitable at sufficient scale." Those are [Speculative] claims, not scaling-law results. The laws describe **pretraining loss on held-out web text**, not "intelligence" or "consciousness." Many downstream tasks scale non-monotonically (Schaeffer 2023). The jump from "loss scales smoothly" to "AGI is a matter of scale" is a bet, not a theorem.

## What the laws actually say
- Loss drops smoothly in a regime.
- Compute-optimal training matches N and D (Chinchilla).
- Exponents drift across modalities, data mixes, and evaluation metrics.
- "Emergent abilities" on discontinuous metrics ≠ true phase transitions.

## What they do not say
- Nothing about consciousness.
- Nothing about AGI timelines.
- Nothing about whether any ceiling exists (L∞ is fitted, not derived).

## Connections
→ [[compute_scaling]]
→ [[emergence_thresholds]]
→ [[005_emergent_abilities]]  (see Schaeffer 2023 caveat there)
← [[predictable_improvement]]

---
Level: L4
Date: 2025-06-21
Updated: 2026-04-18
Tags: #scaling #laws #chinchilla #schaeffer2023 #caveated