# Outputs Directory Structure

This document explains the directory organization and how to use it.

## Overview

```
outputs/
├── prompts/                 # Prompt definitions (versioned code)
├── runs/                    # Run outputs (data)
├── stories/                 # Working directory (latest Stage1)
├── stage2-reports/          # Working directory (latest Stage2)
├── story-index.csv          # Metadata & lineage tracking
└── [reference docs]
```

## Directories

### `prompts/` — Versioned Prompt Definitions

**Purpose:** Source code for prompts. Versioned, tracked in git, auditable.

```
prompts/
├── prompt-01-enrichment/
│   ├── v1.md                # Template for enriching raw stories
│   ├── v2.md (future)       # Refined version
│   └── CHANGELOG.md         # What changed between versions
│
├── prompt-12-dataset-ranking/
│   ├── v1.md (future)       # Template for ranking datasets
│   └── CHANGELOG.md
│
└── prompt-23-exploration/
    ├── v1.md (future)       # Template for validation
    └── CHANGELOG.md
```

**What you do here:**
- Edit prompts to refine them
- Update CHANGELOG with what changed
- Commit to git (prompts are code)

### `runs/` — Isolated Run Outputs

**Purpose:** Timestamped, isolated execution results. Each run is self-contained.

```
runs/
├── stage1_prompt01_v1_20260826/     # Run on 2026-08-26 using prompt-01 v1
│   ├── metadata.json                 # Prompt version, model, timestamp
│   ├── story-01-beavers.md
│   ├── story-02-sar.md
│   └── ... (all 11 stories)
│
├── stage1_prompt01_v2_20260829/     # Re-run with refined prompt-01 v2
│   ├── metadata.json
│   ├── story-01-beavers.md (regenerated)
│   └── ...
│
├── stage2_prompt12_v1_20260827/     # Run using stories from v1 run
│   ├── metadata.json                 # Includes lineage: input_run_dir
│   ├── story-01-beavers-ranking.json
│   └── ... (all 11 stories)
│
└── stage2_prompt12_v2_20260830/     # Re-run with refined prompt-12 v2
    ├── metadata.json
    └── ...
```

**What you do here:**
- Run prompts on stories; output goes here
- Metadata.json tracks which prompt version, model, timestamp
- Do NOT edit files here; they're read-only outputs
- Compare versions to see what changed

**Example metadata.json:**
```json
{
  "stage": 1,
  "prompt_version": "prompt-01-enrichment/v1",
  "model": "claude-opus-5",
  "generated_at": "2026-08-26T14:35:00Z"
}
```

### `stories/` — Working Directory (Latest Stage1)

**Purpose:** Always points to the latest Stage1 output. This is what you read and review.

```
stories/
├── story-01-beavers.md      # Points to latest run
├── story-02-sar.md
└── ... (11 total)
```

**What you do here:**
- Read these to understand the latest enriched stories
- Review for accuracy/domain knowledge fit
- Do NOT edit files here; they're generated
- Update by copying from latest run in `runs/`

**When to update:**
- After you run Prompt-01 with a new version
- Run generates `runs/stage1_prompt01_v2_20260829/`
- Copy those files to `stories/`
- CSV gets updated to track the new version

### `stage2-reports/` — Working Directory (Latest Stage2)

**Purpose:** Always points to the latest Stage2 output.

```
stage2-reports/
├── story-01-beavers-ranking.json
├── story-02-sar-ranking.json
└── ... (11 total)
```

**What you do here:**
- Read these to see dataset rankings for each story
- Review fitness analysis, caveats, quality-of-fit
- Do NOT edit files here; they're generated
- Update by copying from latest run in `runs/`

## Workflow: How It All Connects

### Initial Run (Baseline)

```
1. Run Prompt-01 v1 on all 11 raw stories
   → outputs/runs/stage1_prompt01_v1_20260826/

2. Copy to working dir
   → outputs/stories/ = latest enriched stories

3. CSV records: prompt01_version = v1

4. Run Prompt-12 v1 on stories from step 2
   → outputs/runs/stage2_prompt12_v1_20260827/
   → metadata.json includes: input_run_dir = stage1_prompt01_v1_20260826

5. Copy to working dir
   → outputs/stage2-reports/ = latest dataset rankings

6. CSV records: prompt12_version = v1
```

### Iterating: Refine Prompt-01

```
1. Edit prompts/prompt-01-enrichment/v1.md → v2.md
   Update CHANGELOG

2. Commit to git
   git add prompts/ && git commit -m "Refine Prompt-01 v2: ..."

3. Re-run Prompt-01 v2 on all 11 stories
   → outputs/runs/stage1_prompt01_v2_20260829/
   → metadata.json: prompt_version = "v2"

4. Compare to v1 run
   diff runs/stage1_prompt01_v1_*/story-01-* runs/stage1_prompt01_v2_*/story-01-*

5. If good, update working dir
   cp -r runs/stage1_prompt01_v2_*/story-* stories/
   
6. Update CSV
   prompt01_version = v2, stage1_generated_at = 2026-08-29

7. Stage2 is now out of date (based on v1 stories)
   Re-run Prompt-12 v1 on new v2 stories
   → outputs/runs/stage2_prompt12_v1_new_20260829/
   → metadata.json: input_run_dir = stage1_prompt01_v2_20260829

8. Update stage2-reports/
   cp -r runs/stage2_prompt12_v1_new_*/story-* stage2-reports/

9. Commit everything
   git add prompts/ story-index.csv VERSIONING.md
   git commit -m "Prompt-01 v2: refined enrichment, re-ran Stage 1 & 2"
```

## CSV Tracking

**Key columns:**

```csv
story_id,name,region,persona,
prompt01_version,stage1_generated_at,
prompt12_version,stage2_generated_at,
prompt23_version,stage3_generated_at,
model_version,notes
```

**Example row:**
```csv
01,Beavers in the Arctic,North Slope Borough,Infectious disease researcher,
v1,2026-08-26T14:35:00Z,
v1,2026-08-27T10:20:00Z,
(future),(future),
claude-opus-5,"v1 baseline; Stage 1 & 2 approved by @deweys1"
```

## Reference Documents

- **VERSIONING.md** — Detailed guide to managing versions and re-runs
- **INDEX.md** — Navigation guide
- **README-stories.md** — Process overview
- **STRUCTURE.md** — Why it's organized this way
- **STAGE-1-QUICKSTART.md** — How to run Stage 1

## Quick Reference

| Want to... | Go to... |
|---|---|
| Read latest enriched stories | `stories/` |
| Read latest dataset rankings | `stage2-reports/` |
| Edit a prompt | `prompts/prompt-XX/vN.md` |
| See what changed | `prompts/prompt-XX/CHANGELOG.md` or `git diff` |
| Track lineage | `story-index.csv` or `runs/*/metadata.json` |
| Compare two runs | `diff -u runs/stage1_prompt01_v1_*/story-01-* runs/stage1_prompt01_v2_*/story-01-*` |
| Archive old runs | Move to `runs/_archive_v1/` |

---

**Key Principle:** Prompts are code (versioned in git). Runs are data (timestamped, isolated). Working directories are always-current pointers to the latest run.
