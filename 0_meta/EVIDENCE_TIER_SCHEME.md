# Evidence Tier Scheme

<!-- Canonical reference. Introduced in Phase 1 (2026-04). Enforced in Phase 2. -->

## Why This Exists

The book set was drafted rapidly with minimal review (Phase 0, 2024–2025). Speculation, rhetorical flourishes, and pop-science myths were often written with the same assertive tone as textbook-settled facts. Phase 1 corrections applied tiers retroactively to the most serious cases. Phase 2 will apply them systematically to every chapter via front-matter.

This document defines what each tier means.

## The Three Tiers

### `[Textbook]`

A statement is **Textbook** when it meets all of:

1. Published in multiple standard undergraduate/graduate textbooks in its field.
2. Experimentally or mathematically settled by consensus — no active minority position credibly disputes it.
3. Cited with primary source (DOI, arXiv, ISBN, or equivalent) available on demand.

**Examples:**
- Planck's constant ℏ = 1.055 × 10⁻³⁴ J·s.
- CMB temperature ≈ 2.725 K.
- Chinchilla optimal training ratio is ~20 tokens/parameter (Hoffmann et al. 2022, [arXiv:2203.15556](https://arxiv.org/abs/2203.15556)).
- Silicon lattice constant 0.5431 nm.
- Bathtub Coriolis effect is negligible; Rossby number ~10⁴ (Shapiro 1962).

### `[Cross-disciplinary]`

A statement is **Cross-disciplinary** when:

1. It combines concepts from two or more fields where each ingredient is Textbook in its own field.
2. The combination itself is plausible and supported by working researchers, but not yet textbook consensus.
3. Dissenting positions exist within one or more contributing fields.

**Examples:**
- Information-theoretic approaches to biology (e.g., Jeremy England, 2013 on dissipative adaptation).
- Free-energy principle in neuroscience (Karl Friston).
- Statistical-mechanics framings of language models.
- Sperm whale coda as communicative system (Andreas et al. 2024, *Nat Comm*, [DOI:10.1038/s41467-024-47221-8](https://doi.org/10.1038/s41467-024-47221-8)) — the signal is real, the linguistic structure is argued.

### `[Speculative]`

A statement is **Speculative** when:

1. It is a position held by a minority of active researchers.
2. No experimental or formal result currently settles it.
3. It may be philosophically coherent but is not empirically constrained.
4. It may be the author's own hypothesis not yet peer-reviewed.

**Examples:**
- Panpsychism (Goff 2019; Chalmers 2015).
- Orch-OR microtubule consciousness (Penrose–Hameroff — rejected by Tegmark 2000 decoherence argument).
- Specific Singularity years (2029, 2045, etc.) — Grace et al. 2024 expert survey shows huge variance.
- Simulation hypothesis.
- Omega Point.
- Multiverse interpretations of quantum mechanics.
- Any extrapolative "by year X we will have Y" forecast.

## Required Front-Matter (Phase 2)

Every chapter file will receive at the top:

```html
<!-- Evidence Tier: [Textbook | Cross-disciplinary | Speculative | Mixed]
     Primary sources: <short list>
     Last reviewed: YYYY-MM-DD -->
```

If `Mixed`, the file body must mark each speculative section inline with a `[Speculative]` tag.

## Inline Marker Convention

Use brackets and sentence case:

- `[Textbook]` — rare; usually whole file is Textbook and the header suffices.
- `[Cross-disciplinary]` — when a sentence jumps between two fields.
- `[Speculative]` — before any claim that is not empirically constrained.
- `[Speculative — <reason>]` — preferred when the reason is short (e.g., `[Speculative — no peer-reviewed source]`, `[Speculative — author's hypothesis]`, `[Speculative — philosophy of mind]`).

## Correction Block Convention

For existing content that was published without a tier and is being retrofitted:

```markdown
> **[Correction YYYY-MM — <brief>]** Previous text said <quote>. This is
> wrong/contested because <reason>. Accurate statement: <replacement>.
> Refs: <citations>. Evidence Tier: [<tier>].
```

Do not delete the original unless it is factually false (then delete + note). For speculative claims, keep them but demote the tier.

## What This Scheme Is Not

- **Not a truth score.** Textbook things have been wrong before (aether, phlogiston, continental drift was Speculative in 1920). The tier reflects *current consensus*, not final truth.
- **Not a shaming tool.** Speculative chapters are valuable — they are the author thinking aloud about open frontiers. But the reader must know which tier they are in.
- **Not a substitute for citations.** Every Textbook claim should still cite a primary source.

## Application Order (Phase 2)

1. Foundation (`1_foundation`) — most speculative; most urgent.
2. Domain L9 files — frontier claims need strict marking.
3. Domain L0–L1 files — usually Textbook; quick pass.
4. Domain L2–L8 files — mixed; case by case.
5. Essays, zettel, buddhism, rustlang, ios_voip — domain-specific triage.
