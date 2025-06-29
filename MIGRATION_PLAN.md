# (./books) HA Books Migration Plan

## Phase 1: Backlink Analysis (Current)
1. Extract all internal links from .md files
2. Create complete mapping table
3. Identify cross-references between books

## Phase 2: Directory Structure
```
books/
├── index.md (Master Navigation)
├── 0_meta/
│   └── index.md
├── 1_foundation/
│   ├── index.md
│   ├── HA/
│   ├── HA_HA/
│   └── HA_pattern/
├── 2_physical_emergence/ (13.8 Gya - 4.5 Gya)
│   ├── index.md
│   ├── HA_physics/
│   ├── HA_chemistry/
│   ├── HA_astronomy/
│   ├── HA_energy/
│   └── HA_math/
├── 3_biological_emergence/ (3.8 Gya - 600 Mya)
│   ├── index.md
│   ├── HA_life/
│   ├── HA_evolution/
│   └── HA_consciousness/
├── 4_social_emergence/ (600 Mya - 10 Kya)
│   ├── index.md
│   ├── HA_society/
│   ├── HA_language/
│   ├── HA_psychology/
│   └── HA_religion/
├── 5_civilization_emergence/ (10 Kya - 250 ya)
│   ├── index.md
│   ├── HA_philosophy/ ⭐️ (#1 - Foundation of Civilization)
│   ├── HA_civilization/
│   ├── HA_politics/
│   ├── HA_democracy/
│   ├── HA_empire/
│   ├── HA_ideology/
│   ├── HA_revolution/
│   ├── HA_money/
│   ├── HA_economy/
│   ├── HA_capitalism/
│   ├── HA_trade/
│   └── HA_company/
├── 6_technological_emergence/ (250 ya - Present)
│   ├── index.md
│   ├── HA_technology/
│   ├── HA_algorithms/
│   ├── HA_computer/
│   ├── HA_programming_language/
│   ├── HA_software_engineering/
│   ├── HA_internet/
│   ├── HA_AI/
│   └── HA_megacorp/
├── 7_cultural_emergence/ (Parallel timeline)
│   ├── index.md
│   ├── HA_art/
│   ├── HA_music/
│   ├── HA_writing/
│   ├── HA_cinema/
│   ├── HA_fiction/
│   ├── HA_science_fiction/
│   ├── HA_game/
│   ├── HA_video_game/
│   └── HA_vr/
├── 8_cosmic_futures/
│   ├── index.md
│   ├── HA_space_engineering/
│   ├── HA_solar_system/
│   ├── HA_dyson_sphere/
│   ├── HA_interstellar_civilization/
│   ├── HA_ringworld/
│   └── HA_transportation/
└── essays/
    ├── index.md
    └── sperm_whale_oral_history.md
```

## Phase 3: Navigation System

### Root index.md Features:
1. **Quick Access Panel** (Direct links to key topics)
   - Philosophy (문명의 시작)
   - Physics (우주의 기초)
   - Mathematics (패턴의 언어)
   - Civilization (인간의 구축)
   - HA (메타 프레임워크)

2. **Timeline Navigation**
   - Visual timeline from Big Bang to AI
   - Each era links to its section

3. **Complete Directory**
   - Full hierarchical listing

### Each Section index.md:
```markdown
# [Section Name] - [Timeline]

## Navigation
[← Back to Main](../index.md) | [Timeline View](../index.md#timeline)

## Quick Links
[Important Topic 1] | [Important Topic 2] | [Important Topic 3]

## Books in This Section
1. **Book Name** - Brief description
2. ...

## Related Sections
- Previous: [Section Name]
- Next: [Section Name]
- Related: [Topics]
```

## Phase 4: Backlink Update Process

### Step 1: Create Link Mapping
```bash
# Find all internal links
grep -r "\[.*\]([./]*.*\.md)" . --include="*.md" > all_links.txt

# Generate sed commands for bulk replacement
```

### Step 2: Path Mapping Rules
```
Old Path → New Path
../HA_physics/ → ../../2_physical_emergence/HA_physics/
./HA_philosophy/ → ../5_civilization_emergence/HA_philosophy/
../index.md → ../../index.md
```

### Step 3: Update Links
- Use sed/perl for bulk replacements
- Verify each replacement
- Test all links

## Phase 5: Verification
1. Check all internal links resolve
2. Verify navigation works at each level
3. Test quick access shortcuts
4. Ensure breadcrumbs are correct

## Rollback Plan
- Keep full backup of current structure
- Document all changes
- Ability to revert if issues found