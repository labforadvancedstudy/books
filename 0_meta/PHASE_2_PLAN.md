# Phase 2 Plan — Knowledge Tree Restructure + Gap Fill

<!-- Drafted 2026-04-18. Status: proposal for discussion. -->
<!-- Context: Phase 1 (corrections + Evidence Tier scheme) merging in parallel. -->

## Stated Goal

From user intent (2026-04):

> "인류가 알고 있는 모든 지식을 계층으로 분리하고 그러면 우리가 뭘 모르고 있는지 어느 분야가 빠져 있는지를 알려는 시도. 얕더라도 인류가 알고 있는 모든 지식을 계층적으로 모두 작성하는게 목표."
>
> - 모든 지식을 계층적으로 정리하고 각각을 교과서화한다
> - 트리구조로 쌓으면 (N차원 지식을 2D 트리로 압축) 빠진 분야가 비어있게 드러난다
> - speculation은 정확히 라벨링한다 (Phase 1에서 완료)

Translation: comprehensive, shallow-but-complete, hierarchy-first. Gaps must be **visibly empty**, not hidden.

## The N-dim → 2D Problem

Knowledge is N-dimensional. At minimum:

| Axis | Range |
|---|---|
| **Scale** | quantum → particle → atom → molecule → cell → organism → society → civilization → cosmic |
| **Time** | prehistory → history → present → near-future → far-future |
| **Epistemology** | observed → measured → theoretical → speculative → metaphysical |
| **Method** | empirical → mathematical → phenomenological → narrative → normative |
| **Pragmatism** | pure theory → applied → engineering → craft → art |
| **Subject** | self → other → society → world → cosmos |
| **Abstraction** | concrete → pattern → principle → meta-principle |

The current 2D projection is `(Emergence Level × HA_domain × L0-L9)` ≈ `Scale × Subject × Abstraction`. This misses the other axes entirely. But adding axes makes the tree unviewable.

**Design principle**: Keep the 2D (emergence × domain × level) projection. For each *other* axis, encode it inside the chapter file via front-matter + tagged sections. This lets humans browse by the dominant 2D axis and agents filter by any dimension.

## Proposed 2D Tree (revised)

```
0_meta/                      ← framework, tier scheme, gap index
1_foundation/                ← HA theory itself (L0-L9)
2_physical_emergence/        ← physics, chemistry, math, astronomy
3_biological_emergence/      ← life, evolution, ecology, consciousness, neuroscience
4_social_emergence/          ← language, linguistics, psychology, religion, society
5_civilization_emergence/    ← civilization, politics, economy, law, philosophy, ...
6_technological_emergence/   ← AI, computer, algorithms, internet, SW engineering, ...
7_cultural_emergence/        ← art, cinema, fiction, music, writing, game, VR
8_cosmic_futures/            ← dyson sphere, ringworld, interstellar, space engineering
9_earth_systems/      [NEW]  ← geology, meteorology, oceanography, climate science
10_material_world/    [NEW]  ← materials, manufacturing, energy, agriculture, food
11_applied_sciences/  [NEW]  ← medicine, public health, law, education, sports
12_human_life/        [NEW]  ← childhood, development, gender, sexuality, death, rites
13_engineering_disciplines/ [NEW]  ← mechanical, electrical, civil, aerospace, chemical eng
```

Rationale for 9–13: current sections are biased toward **emergence phenomena** (cool, speculative, philosophical). Everyday, worked-out, textbook-heavy disciplines (medicine, law, agriculture, mechanical engineering) have no home. Without them the tree cannot claim to be "all human knowledge."

### Alternative: keep flat 0–8, distribute gaps into existing sections

- Earth systems → `2_physical_emergence/HA_earth_systems/` etc.
- Medicine → `3_biological_emergence/HA_medicine/`
- Law → `5_civilization_emergence/HA_law/`
- Mechanical engineering → `6_technological_emergence/HA_mechanical_engineering/`

Advantage: no disruption to existing top-level numbering.
Disadvantage: hides the scale of missing content. 20+ gaps sprinkled across 8 sections won't feel like "we're missing a lot."

**Recommendation**: split approach. Create top-level 9/10/11/12/13 for truly emerged-independent disciplines. For disciplines that clearly fit existing sections (medicine under bio, law under civ, linguistics under social), add them as HA_* subdirectories.

## Gap Inventory (20+ missing fields)

Discovered by comparing existing tree to standard university library classification (LC + Dewey cross-referenced):

### Critical gaps (missing entirely)

| # | Domain | Proposed location | Why critical |
|---|---|---|---|
| 1 | **Geology / Earth sciences** | `9_earth_systems/HA_geology/` | Foundation of all surface life; plate tectonics, rock cycle |
| 2 | **Meteorology / climate** | `9_earth_systems/HA_climate/` | Civilization-scale pressure in the 21st century |
| 3 | **Oceanography** | `9_earth_systems/HA_ocean/` | Covers 70% of planet |
| 4 | **Materials science** | `10_material_world/HA_materials/` | Every artifact depends on this |
| 5 | **Manufacturing / production systems** | `10_material_world/HA_manufacturing/` | Toyota Production System, lean, supply chains |
| 6 | **Agriculture / food systems** | `10_material_world/HA_agriculture/` | Civilizational substrate (12,000 yr old) |
| 7 | **Medicine / clinical practice** | `11_applied_sciences/HA_medicine/` | Largest applied bio field |
| 8 | **Public health / epidemiology** | `11_applied_sciences/HA_public_health/` | COVID showed this is underrated |
| 9 | **Law / jurisprudence** | `11_applied_sciences/HA_law/` | Current tree has politics but no legal theory |
| 10 | **Education as a system** | `11_applied_sciences/HA_education/` | How knowledge propagates (meta-gap) |
| 11 | **Mechanical engineering** | `13_engineering/HA_mechanical/` | SW is heavily covered; physical engineering absent |
| 12 | **Electrical engineering** | `13_engineering/HA_electrical/` | Power, control, analog |
| 13 | **Civil engineering / architecture / urban planning** | `13_engineering/HA_civil_architecture/` | Built environment |
| 14 | **Aerospace engineering** | `13_engineering/HA_aerospace/` | Prerequisite to 8_cosmic_futures |
| 15 | **Chemical engineering / process engineering** | `13_engineering/HA_chemical_eng/` | Industrial chemistry at scale |
| 16 | **Ecology** (distinct from HA_life) | `3_biological_emergence/HA_ecology/` | Ecosystems, biodiversity, trophic cascades |
| 17 | **Neuroscience** (distinct from HA_consciousness) | `3_biological_emergence/HA_neuroscience/` | Empirical brain, not philosophy |
| 18 | **Linguistics** (distinct from HA_language philosophy) | `4_social_emergence/HA_linguistics/` | Chomskyan, typological, historical |
| 19 | **Cognitive science** | `3_biological_emergence/HA_cognitive_science/` | Between neuro and consciousness |
| 20 | **Cryptography** | `6_technological_emergence/HA_cryptography/` | Foundational to current society |
| 21 | **Accounting / bookkeeping** | `5_civilization_emergence/HA_accounting/` | Money mechanics at firm level |
| 22 | **Sports / physical training / body** | `12_human_life/HA_body/` | Human capability, Olympics, martial arts |
| 23 | **Childhood / development** | `12_human_life/HA_development/` | Distinct life phase |
| 24 | **Gender / sexuality** | `12_human_life/HA_gender_sexuality/` | Absent entirely |
| 25 | **Death / grief / rites of passage** | `12_human_life/HA_mortality/` | Religion touches; not systematized |
| 26 | **Energy systems** (distinct from physics) | `10_material_world/HA_energy/` | Generation, distribution, storage |
| 27 | **Transportation systems** (distinct from cosmic) | `10_material_world/HA_transport_systems/` | Ground/sea/air at industrial scale |

### Partially-present (exists but thin)

| # | Domain | Current location | What's missing |
|---|---|---|---|
| A | Chemistry | `2_physical_emergence/HA_chemistry/` | Biochemistry, organic chem, analytical chem depth |
| B | Psychology | `4_social_emergence/HA_psychology/` | Clinical, developmental, social psych thin |
| C | Economics | `5_civilization_emergence/HA_economy/` | Macro dense, micro/finance/game-theory thin |
| D | Art | `7_cultural_emergence/HA_art/` | Art history, specific movements missing |
| E | Music | `7_cultural_emergence/HA_music/` | Music theory proper, genres by region missing |

## Each Gap Gets L0-L9 Stub

Every new HA_* directory gets 10 files: `L0_Index.md`, `L1_*.md` … `L9_*.md`. Each file is short (300–1000 words), textbook-shallow, with Evidence Tier front-matter. Over time, depth accumulates.

### L0-L9 Template (for gap domains)

```markdown
<!-- Evidence Tier: [Textbook | Cross-disciplinary | Speculative | Mixed]
     Primary sources: <key refs>
     Last reviewed: YYYY-MM-DD -->

# Level N: <Chapter Title>
*<one-line subtitle>*

## What this level covers
<2-3 sentences, what question this level answers>

## Core ideas
<3-7 bullets, textbook content>

## Key people / moments
<3-5 names or dates, so readers can follow threads>

## What is contested / open
<1-3 bullets, marked [Speculative]>

## Connections
<links to adjacent chapters in same domain and cross-domain>

## Further reading
<3-5 primary sources with DOIs/ISBNs>
```

### Per-Level Semantics (applied to new gap domains)

| Level | Semantic | Example (for HA_medicine) |
|---|---|---|
| L0 | Index / map | Table of contents for medicine |
| L1 | Phenomenal / child's view | "I'm sick. The doctor helps." |
| L2 | Measure / quantify | Vital signs, lab values, imaging |
| L3 | Mechanism | Disease processes, physiology |
| L4 | System integration | Organ systems, homeostasis |
| L5 | Clinical practice | Diagnosis, treatment, evidence-based medicine |
| L6 | Meta / philosophy | Medical ethics, what "health" means |
| L7 | Institutional | Hospitals, insurance, regulation |
| L8 | Civilization-scale | Public health, global health, pandemics |
| L9 | Frontier / open | Aging, longevity, consciousness, end-of-life |

Applied per-domain with the same template. For each new HA_*, define its L1-L9 semantic grid before writing.

## Front-matter Application Strategy (Phase 2 execution)

- Start with 0_meta and 1_foundation (already small, Phase 1 touched most files).
- Then every L9 file (per L9_TEMPLATE_WARNING.md — many need rewrite).
- Then L0 indexes (small, high-leverage for navigation).
- Then L1-L8 in parallel by section.

Automation: script that scans each `.md` file, detects missing front-matter, inserts default `[Textbook]` tier for now. Human pass to downgrade where needed.

## Korean Filename Unification

Current inconsistency:
- `L5_Fields_and_Waves.ko.md` (new style — keeps English stem)
- `L9_근본과_신비.md`, `L9_우주_시장.md` (old style — Korean stem, no `.ko.md`)

Decision: **keep English stem + `.ko.md` suffix**. Reasons:
1. Translation pairs sort together in file browsers.
2. Cross-references between en and ko are automatic (change `.md` → `.ko.md`).
3. Git blame survives rename.

Migration: 15 known old-style files get `git mv` to new style; update internal links. Script to detect broken links post-rename.

## Phased Execution (inside Phase 2)

### Phase 2A: Tree scaffolding (1-2 hours)
- Create `9_earth_systems/`, `10_material_world/`, `11_applied_sciences/`, `12_human_life/`, `13_engineering/`
- Stub `000_index.md` for each
- Update top-level `index.md` and `HA_master_index.md`
- Commit + PR

### Phase 2B: Gap L0 stubs (3-5 hours)
- Each of 27 new HA_* directories gets `L0_Index.md` with 2D semantic grid, planned L1-L9 titles
- Commit + PR (visible gap inventory)

### Phase 2C: Gap L1-L2 stubs (depth 0) (10-20 hours)
- Each HA_* gets L1 and L2 filled in, 300-500 words each
- "Shallow but present"
- Commit + PR

### Phase 2D: L9 template rewrite (5-10 hours)
- 15 flagged L9 files rewritten to domain-specific open-problem form
- Per L9_TEMPLATE_WARNING.md structure
- Commit + PR

### Phase 2E: Front-matter pass (2-3 hours + agent)
- Script adds default `[Textbook]` header to every `.md`
- Manual review of `[Speculative]`/`[Cross-disciplinary]` downgrades
- Commit + PR

### Phase 2F: ko filename unification (1 hour + script)
- `git mv` old-style Korean filenames to `{English}.ko.md`
- Update internal links (ripgrep + sed)
- Commit + PR

### Phase 2G: Gaps index publication (1 hour)
- `0_meta/KNOWLEDGE_GAPS.md` — canonical "what's still empty" document
- Cross-references every HA_* whose L1-L9 is <50% filled
- Updates auto-generated on commit (script)

## Scope Estimate

| Phase | Files created/touched | Time estimate | Reviewer |
|---|---|---|---|
| 2A scaffolding | ~30 files | 1-2h | user |
| 2B L0 stubs | ~27 files | 3-5h | user |
| 2C L1-L2 depth | ~54 files | 10-20h | user + agent |
| 2D L9 rewrites | ~15 files | 5-10h | user |
| 2E front-matter | ~950 files (scripted) | 2-3h | spot check |
| 2F ko rename | ~15 files | 1h | user |
| 2G gaps index | ~1 file (auto-gen) | 1h | user |
| **Total** | **~1100 files touched** | **22-40h human + agent** | |

## Risks

1. **Scope creep.** 27 new domains at L0-L2 depth ≈ 54 new files, each 300-500 words. Non-trivial. Suggest shipping 2A (scaffolding) alone as a first small PR to validate direction.
2. **Ko translation lag widens.** New en content will outpace ko. Track in TRANSLATION_STATUS.md.
3. **L1-L9 semantic drift across domains.** What "Level 3: Mechanism" means for chemistry is not what it means for law. Mitigation: per-domain L-semantic grid documented in each L0.
4. **Speculation creep back in.** Once stubs are filled in iterations, speculation can accrete. Evidence Tier header + git pre-commit lint is the counter.

## Open Questions for User

1. **Top-level sections 9-13**: create new (proposed), or embed gaps inside existing 0-8? Recommendation: create new, to make scale visible.
2. **L9_TEMPLATE_WARNING.md**: rewrite 15 flagged L9 files in Phase 2D, or leave flagged and rewrite case-by-case later? Recommendation: batch rewrite in 2D.
3. **ko parity**: Phase 2 focuses on en; ko is 48% behind. Accept widening gap, or gate en content on ko catchup? Recommendation: accept; track in status.
4. **Gap priority**: of 27 new domains, which 5 matter most for v1? Suggestions: medicine, law, ecology, materials, energy.
5. **Execution pace**: all-at-once push (one big Phase 2 PR), or 7 sub-PRs 2A-2G? Recommendation: 7 sub-PRs for reviewability.

## Cross-references

- `0_meta/EVIDENCE_TIER_SCHEME.md` — tier definitions applied in 2E
- `0_meta/L9_TEMPLATE_WARNING.md` — flagged L9 files for 2D rewrite
- `TRANSLATION_STATUS.md` — ko tracking for 2F
