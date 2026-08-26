# Prompt Versioning & Re-runs

This document explains how to manage and re-run prompts as you iterate on the pipeline.

## Directory Structure

```
outputs/
├── prompts/                          # Prompt definitions (versioned)
│   ├── prompt-01-enrichment/
│   │   ├── v1.md
│   │   ├── v2.md (future)
│   │   └── CHANGELOG.md
│   ├── prompt-12-dataset-ranking/
│   │   ├── v1.md (future)
│   │   └── CHANGELOG.md
│   └── prompt-23-exploration/
│       ├── v1.md (future)
│       └── CHANGELOG.md
│
├── runs/                            # Run outputs (timestamped, isolated)
│   ├── stage1_prompt01_v1_20260826/
│   │   ├── metadata.json
│   │   ├── story-01-beavers.md
│   │   └── ... (all 11 stories)
│   └── stage2_prompt12_v1_20260827/
│       ├── metadata.json
│       ├── story-01-beavers-ranking.json
│       └── ... (all 11 stories)
│
├── stories/                         # Working copy (points to latest stage1)
│   ├── story-01-beavers.md
│   └── ... (11 total)
│
├── stage2-reports/                  # Working copy (points to latest stage2)
│   ├── story-01-beavers-ranking.json
│   └── ... (11 total)
│
└── story-index.csv                  # Tracks versions and lineage
```

## Workflow: Iterating on Prompts

### When You Refine Prompt-01 (Enrichment)

**1. Save the new version:**
```bash
cp outputs/prompts/prompt-01-enrichment/v1.md outputs/prompts/prompt-01-enrichment/v2.md
# Edit v2.md with your refinements
```

**2. Update CHANGELOG:**
```bash
# Edit outputs/prompts/prompt-01-enrichment/CHANGELOG.md
# Add section for v2 with what changed
```

**3. Run the prompt on all 11 stories:**
```bash
# Claude Code: Run Prompt-01 v2 on each story
# Save outputs to: outputs/runs/stage1_prompt01_v2_20260829/
```

**4. Update working directory:**
```bash
# Copy or symlink latest run to outputs/stories/
cp -r outputs/runs/stage1_prompt01_v2_20260829/* outputs/stories/
```

**5. Update CSV:**
```bash
# Edit outputs/story-index.csv
# Set prompt01_version = v2 for all stories
# Set stage1_generated_at = 2026-08-29
```

### When You Refine Prompt-12 (Dataset Ranking)

**1. Save the new version:**
```bash
cp outputs/prompts/prompt-12-dataset-ranking/v1.md outputs/prompts/prompt-12-dataset-ranking/v2.md
# Edit v2.md with refinements
```

**2. Update CHANGELOG:**
```bash
# Edit outputs/prompts/prompt-12-dataset-ranking/CHANGELOG.md
# Add section for v2
```

**3. Run on all 11 enriched stories:**
```bash
# Claude Code: Run Prompt-12 v2 on each story from outputs/stories/
# Each story's enrichment is input
# Output: Dataset Ranking Report (JSON or markdown)
# Save to: outputs/runs/stage2_prompt12_v2_20260830/
```

**4. Update working directory:**
```bash
cp -r outputs/runs/stage2_prompt12_v2_20260830/* outputs/stage2-reports/
```

**5. Update CSV:**
```bash
# Edit outputs/story-index.csv
# Set prompt12_version = v2 for all stories
# Set stage2_generated_at = 2026-08-30
```

## Run Metadata

Each run directory contains a `metadata.json` file:

```json
{
  "stage": 1,
  "prompt_version": "prompt-01-enrichment/v1",
  "model": "claude-opus-5",
  "generated_at": "2026-08-26T14:35:00Z",
  "model_settings": {
    "temperature": 0.7,
    "max_tokens": 8000
  },
  "input_version": null,
  "notes": "Initial enrichment run on all 11 stories"
}
```

For Stage 2+, include lineage:

```json
{
  "stage": 2,
  "prompt_version": "prompt-12-dataset-ranking/v1",
  "model": "claude-opus-5",
  "generated_at": "2026-08-27T10:20:00Z",
  "input_stage": 1,
  "input_prompt_version": "prompt-01-enrichment/v1",
  "input_run_dir": "stage1_prompt01_v1_20260826",
  "notes": "First Stage 2 run testing prompt-12 v1"
}
```

## Comparing Versions

### See what changed in a prompt:
```bash
git diff outputs/prompts/prompt-01-enrichment/v1.md outputs/prompts/prompt-01-enrichment/v2.md
```

### See which stories used which version:
```bash
grep "prompt01_version" outputs/story-index.csv
```

### Compare outputs from two runs:
```bash
diff -u outputs/runs/stage1_prompt01_v1_20260826/story-01-beavers.md \
         outputs/runs/stage1_prompt01_v2_20260829/story-01-beavers.md
```

## CSV Columns for Version Tracking

```csv
story_id,name,region,persona,
prompt01_version,stage1_generated_at,stage1_passed,
prompt12_version,stage2_generated_at,stage2_passed,
prompt23_version,stage3_generated_at,stage3_passed,
model_version,notes
```

**Example:**
```csv
01,Beavers in the Arctic,North Slope Borough,Infectious disease researcher,
v1,2026-08-26T14:35:00Z,true,
v1,2026-08-27T10:20:00Z,true,
(future),(future),(future),
claude-opus-5,"Initial run; Stage 1 & 2 passed validation"
```

## LLM Swapping

When you later test other LLMs (GPT-4, etc.), track it:

```bash
# New prompt version optimized for GPT-4
cp outputs/prompts/prompt-01-enrichment/v2.md outputs/prompts/prompt-01-enrichment/v2-gpt4.md
# Run with GPT-4
# Track in CSV: model_version = "claude-opus-5" vs "gpt-4"
```

## Archiving & Cleanup

Keep old runs for reference, but don't clutter working directories:

```bash
# Archive old runs
mkdir -p outputs/runs/_archive_v1
mv outputs/runs/stage1_prompt01_v1_* outputs/runs/_archive_v1/

# Working directories (stories/, stage2-reports/) always point to latest
```

## Automated Re-runs (Future)

When ready, a script could automate this:

```bash
# Pseudo-code (not yet implemented)
python scripts/run-prompt.py \
  --stage 1 \
  --prompt-version v2 \
  --model claude-opus-5 \
  --input-dir ./outputs/stories \
  --output-dir ./outputs/runs
```

For now, manual runs + careful CSV tracking.

---

## Summary

- **Prompts:** Versioned in `prompts/` with CHANGELOG
- **Runs:** Timestamped, isolated in `runs/`, linked from working dirs
- **Lineage:** Tracked in CSV and metadata.json
- **Iteration:** Update prompt → run → update CSV → compare outputs
- **LLM swapping:** Add model_version column to track
