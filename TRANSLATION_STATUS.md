# Translation Status & Workflow

## Current Status

### Korean (ko) - 한국어
- **Structure Created**: Full directory structure mirroring English
- **Documents Completed**: 
  - 1 foundation document: HA_is_everything.ko.md (originally Korean, translated to English)
  - 10 physics documents: Complete HA_physics book (Index + L1-L9)
  - 10 philosophy documents: Complete HA_philosophy book (Index + L1-L9) ✨ NEW

### Directory Structure
```
books/
├── [English content - default]
├── ko/                    # Korean translations
│   ├── 0_meta/
│   ├── 1_foundation/
│   │   └── HA_is_everything.ko.md
│   ├── 2_physical_emergence/
│   │   └── HA_physics/
│   │       ├── HA_Physics_Index.ko.md
│   │       ├── L1_Things_Fall_Down.ko.md
│   │       ├── L2_Measuring_Motion.ko.md
│   │       ├── L3_Force_and_Energy.ko.md
│   │       ├── L4_Conservation_Laws.ko.md
│   │       ├── L5_Fields_and_Waves.ko.md
│   │       ├── L6_Relativity.ko.md
│   │       ├── L7_Quantum_Rules.ko.md
│   │       ├── L8_Quantum_Fields.ko.md
│   │       └── L9_The_Edge.ko.md
│   ├── 3_biological_emergence/
│   ├── 4_social_emergence/
│   ├── 5_civilization_emergence/
│   ├── 6_technological_emergence/
│   ├── 7_cultural_emergence/
│   ├── 8_cosmic_futures/
│   └── essays/
└── [other language folders in future]
```

## Translation Workflow

### Commands
- **'한글 번역'** - Start Korean translation work
- **'한글 업데이트'** - Update Korean translations (check dates, retranslate if needed)

### Rules
1. Default language is English (no language marker in filename)
2. Korean files use `.ko.md` extension
3. Maintain exact directory structure in language folders
4. Translate one file at a time (large volume of content)
5. Do NOT translate zettel files
6. For updates: Compare file dates and retranslate if English is newer

### Next Steps for Korean Translation
Priority order for remaining translations:
1. **1_foundation/HA/** - Core HA book
2. **5_civilization_emergence/HA_philosophy/** - Philosophy (문명의 시작)
3. **2_physical_emergence/HA_math/** - Mathematics
4. **3_biological_emergence/HA_consciousness/** - Consciousness
5. Other books following emergence timeline

## Adding New Languages
To add a new language (e.g., Japanese):
1. Create language folder: `mkdir -p ja/{all_subdirectories}`
2. Use appropriate file extension: `.ja.md`
3. Follow same structure and workflow