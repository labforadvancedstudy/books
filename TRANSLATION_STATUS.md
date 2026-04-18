# Translation Status & Workflow

<!-- Last updated: 2026-04-18 (Phase 1H) -->

## Current Status

### Korean (ko) — 한국어

Actual file count per section (as of 2026-04-18):

| Section | Ko files | English files | Coverage |
|---|---:|---:|---|
| 1_foundation | 60 | ~90 | Partial |
| 2_physical_emergence | 99 | ~120 | Near-complete |
| 3_biological_emergence | 90 | ~110 | Near-complete |
| 4_social_emergence | 73 | ~100 | Partial |
| 5_civilization_emergence | 135 | ~180 | Partial |
| 6_technological_emergence | 0 | ~150 | **Not started** |
| 7_cultural_emergence | 0 | ~120 | **Not started** |
| 8_cosmic_futures | 0 | ~80 | **Not started** |

Total ko: 457 `.md` files; total en: ~950 files. Ko coverage ≈ 48%.

### Filename Conventions (inconsistent — Phase 2 will unify)

Two conventions currently coexist:

1. `Lx_English_Name.ko.md` — newer convention (e.g. `L5_Fields_and_Waves.ko.md`).
2. `Lx_한글_이름.md` — older convention (e.g. `L9_경계의_붕괴_현대수학의_최전선.md`, `L9_근본과_신비.md`, `L9_우주_시장.md`).

Phase 2 will standardize on `.ko.md` with the English filename stem (so translation pairs sort together).

### Directory Structure

```
books/
├── [English content — default]
├── 0_meta/
├── 1_foundation/
├── 2_physical_emergence/
├── 3_biological_emergence/
├── 4_social_emergence/
├── 5_civilization_emergence/
├── 6_technological_emergence/
├── 7_cultural_emergence/
├── 8_cosmic_futures/
├── buddhism/                # Korean originals
├── essays/                  # Mixed languages
├── ios_voip/                # Technical notes
├── rustlang/                # Technical notes
└── ko/                      # Korean translations (mirrors section dirs)
    ├── 0_meta/
    ├── 1_foundation/
    │   ├── HA_is_everything.ko.md
    │   └── HA/L*.md (partial)
    ├── 2_physical_emergence/
    │   └── HA_physics/L1–L9.ko.md (complete)
    ├── 3_biological_emergence/
    ├── 4_social_emergence/
    └── 5_civilization_emergence/
```

## Translation Workflow

### Commands
- **'한글 번역'** — Start Korean translation work.
- **'한글 업데이트'** — Check mtime on English original and retranslate if newer.

### Rules

1. Default language is English. No language marker in filename.
2. Korean translations use `.ko.md` extension.
3. Maintain exact directory structure in language folders.
4. Translate one file at a time.
5. **Do NOT translate `zettel/` files.** They are working notes, not publication content.
6. For updates, compare mtime and retranslate if English is newer.
7. **[New, Phase 1H]** Every translation must preserve Evidence Tier markers (`[Textbook]`, `[Cross-disciplinary]`, `[Speculative]`) and `[Correction YYYY-MM]` blocks exactly. Do not paraphrase them away.

### Priority Order for Remaining Korean Translation

1. **6_technological_emergence/HA_AI/** — 0% translated; high-traffic content.
2. **7_cultural_emergence/HA_writing/** and HA_cinema/ — 0% translated.
3. **8_cosmic_futures/** — 0% translated; short section.
4. **1_foundation/HA/L4–L8** — Core HA theory, gaps remain.
5. **4_social_emergence/HA_psychology/** and HA_language/ — fill gaps.

## Adding New Languages

To add a new language (e.g., Japanese):

1. Create language folder: `ja/` mirroring top-level sections.
2. Use file extension: `.ja.md`.
3. Follow same workflow and Evidence Tier preservation rules.

## Related

- `0_meta/EVIDENCE_TIER_SCHEME.md` — Textbook / Cross-disciplinary / Speculative definitions.
- `0_meta/L9_TEMPLATE_WARNING.md` — L9 pattern audit from Phase 1G.
