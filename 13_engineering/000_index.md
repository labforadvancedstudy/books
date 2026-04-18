# 13. Engineering

<!-- Evidence Tier: Textbook (expected once filled)
     Status: Phase 2A scaffolding — subdirectories to follow in Phase 2B
     Added: 2026-04-18 -->

## What this section covers

The classical engineering disciplines — mechanical, electrical, civil, aerospace, chemical. These are the knowledge bodies that translate physics into reliable, buildable, scalable systems.

The original tree lumped everything under `HA_technology/` and `HA_software_engineering/`, as though bridges, jet engines, and chemical plants were minor subfields of general technology. They are not. Each has centuries of accumulated design practice, distinct mathematical tool sets, distinct failure modes, and distinct professional cultures.

## Planned HA_* subdirectories (Phase 2B)

| Path | Domain | L9 frontier |
|---|---|---|
| `HA_mechanical/` | Statics, dynamics, thermodynamics as applied, machine design | Programmable-material mechanisms, morphing structures |
| `HA_electrical/` | Power electronics, signal processing, control systems, RF | Quantum-classical interface, ultra-low-power compute |
| `HA_civil_architecture/` | Structures, materials, construction, urban design | Self-healing concrete, seismic resilience, off-planet construction |
| `HA_aerospace/` | Aerodynamics, propulsion, orbital mechanics, flight systems | Reusable heavy lift, atmospheric-to-orbit spaceplanes |
| `HA_chemical_eng/` | Reaction engineering, unit operations, process scale-up | Electrified chemistry (no-combustion industrial heat) |

## Why this exists as a separate top-level

- These are **design sciences** — knowledge organized around what you must know to build something that works, which is different from knowledge organized around how the universe works.
- They have a shared grammar: tolerances, margins, failure modes, codes and standards, iterative refinement with test. That grammar does not show up in `HA_physics` or `HA_computer`.
- Engineering is the place where `2_physical_emergence` (physics), `10_material_world` (materials, energy), and `5_civilization_emergence` (cost, labor, regulation) actually meet and produce artifacts.

## Relationship to other sections

- `HA_software_engineering/` stays in `6_technological_emergence/` because software has its own character (no physical failure, copyable, compositional). Classical engineering is the complement.
- `HA_space_engineering/` in `8_cosmic_futures/` overlaps `HA_aerospace/` but at the frontier: aerospace covers today's aviation and space, space engineering covers hypothetical megastructures.
- `HA_manufacturing/` in `10_material_world/` is close but different: manufacturing = the production system, engineering = the design discipline.

## Evidence tier philosophy for this section

- L1-L7: **[Textbook]** — mature engineering sciences, well-validated.
- L8: **[Cross-disciplinary]** — engineering ethics, systems-of-systems, infrastructure policy.
- L9: **[Speculative]** — frontier technologies that may or may not ever scale.

## Cross-references

- `0_meta/PHASE_2_PLAN.md` — why this section was added
- `0_meta/EVIDENCE_TIER_SCHEME.md` — tier definitions
- `2_physical_emergence/HA_physics/` — underlying physics
- `6_technological_emergence/HA_software_engineering/` — software's own engineering discipline
- `8_cosmic_futures/HA_space_engineering/` — speculative megascale engineering
- `10_material_world/HA_manufacturing/` — production systems that realize engineering designs
