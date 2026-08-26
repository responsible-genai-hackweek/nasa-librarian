# User Stories: Refinement → LLM+MCP → Validation Pipeline

This directory contains the structured pipeline for refining raw user stories into testable, validatable dataset queries.

## Directory Structure

```
outputs/
├── README-stories.md                    # This file
├── story-index.csv                      # Dashboard: status of all 11 stories
├── stage-0-information-architecture.md  # Template for enrichment
├── stage-1-prompt-template.md          # Template for Stage 1 LLM queries
│
├── stories/
│   ├── story-01-beavers.md
│   ├── story-02-sar.md
│   ├── story-03-algal-blooms.md
│   ├── ... (11 total)
│
├── stage-1-prompts/
│   ├── prompt-01-beavers.txt
│   ├── prompt-02-sar.txt
│   ├── ... (one per story; used to query earthdata-mcp)
│
└── stage-1-responses/
    ├── response-01-beavers.txt
    ├── response-02-sar.txt
    └── ... (LLM+MCP output; compared against story spec for validation)
```

---

## The Pipeline

### Stage 0: Enrichment (Complete)

**Input:** Raw user story from issue #3 (thin, tacit domain knowledge)

**Process:** Structured enrichment using an information architecture:
- Persona context (role, stakes, expertise)
- Raw problem statement
- Six slots (unit of analysis, time window, cadence, variable meaning, effect size, latency)
- Gotchas & traps (where naive approaches fail)
- Dataset suitability criteria ("Would work" / "Wouldn't work")

**Output:** Enriched story markdown in `stories/story-NN.md`

**Deliverable:** `/home/jovyan/nasa-librarian/outputs/stage-0-issue-body.md` (11 stories enriched)

---

### Stage 1: LLM + earthdata-mcp Query (In Progress)

**Input:** Enriched story (from Stage 0)

**Process:**
1. **Parse constraints:** Extract the six slots into machine-readable form
2. **Query earthdata-mcp:** Search for collections matching variables, region, time window, resolution
3. **Validate candidates:** For each dataset returned, check:
   - Does it fit the specification?
   - Which gotchas does it hit or avoid?
   - Rank by fitness, not relevance
4. **Refuse appropriately:** If all candidates fail an axis, say so

**Prompt template:** `/home/jovyan/nasa-librarian/outputs/stage-1-prompt-template.md`

**Instance prompts:** `stage-1-prompts/prompt-NN-*.txt` (one per story)

**Output:** Structured response (JSON) in `stage-1-responses/response-NN-*.txt`

**Validation:** Compare LLM's picks against Stage 0 predictions. Did it find Landsat? Did it correctly reject SMAP? Did it surface the naive-picker trap?

---

### Stage 2: Exploratory Data Analysis (Future)

**Input:** Top 1–3 candidate datasets (from Stage 1 validation)

**Process:** Download and test against pilot region
- Retrieve actual granules
- Apply recommended algorithms (NDVI time series, InSAR stacking, spectral decomposition)
- Compare to available ground truth
- Document successes and failures

**Output:** Hand-authored readiness records (the operand) + findings (can this approach actually work?)

---

## How to Use This Pipeline

### For Researchers / Domain Experts

1. **Review enriched stories:** Open `stories/story-NN.md` and check the Stage 0 enrichment
2. **Correct or clarify:** Edit the six slots if domain knowledge suggests changes
3. **Run Stage 1:** Provide the LLM + earthdata-mcp with the Stage 1 prompt
4. **Validate Stage 1:** Read the response; check if the datasets make sense
5. **Advance to Stage 2:** If validation passes, schedule exploratory data work

### For Software / Data Engineers

1. **Generate Stage 1 prompts:** Template is in `stage-1-prompt-template.md`; instantiate for each story
2. **Run LLM+MCP queries:** Use Claude Code with earthdata-mcp tools to generate responses
3. **Parse responses:** Extract structured JSON from Stage 1 responses
4. **Build validation suite:** Script to compare Stage 1 picks against Stage 0 "Would Work" / "Wouldn't Work"
5. **Hand to Aimee:** Pass validated dataset lists to Aimee for readiness record authoring

### For Project Managers

1. **Check dashboard:** `story-index.csv` shows status across all 11 stories
2. **Track blockers:** Which stories are ready for which stage?
3. **Identify quick wins:** Stories with high confidence and clear dataset matches
4. **Flag hard cases:** Stories with scale traps, latency challenges, or data gaps

---

## Key Concepts

### Six Slots

Every enriched story must specify these explicitly:

| Slot | Why It Matters |
|---|---|
| **Unit of Analysis** | Determines minimum spatial resolution |
| **Time Window** | Determines minimum archive length |
| **Cadence** | Determines whether revisit frequency is adequate |
| **Variable Meaning** | Determines semantic match to datasets |
| **Effect Size** | Determines measurement uncertainty threshold |
| **Latency** | Determines real-time vs. retrospective capability |

### Gotchas & Traps

Each story identifies where naive satellite-data users fail:
- Confusing air temperature with ground temperature
- Picking a dataset that's too coarse
- Mistaking keyword relevance for fitness
- Ignoring seasonal or regional caveats

The LLM's job in Stage 1 is to surface these traps **before** the user wastes time downloading.

### Refusal as Success

The librarian's job is not always to find an answer. Sometimes the right response is:
- "Nothing in NASA's holdings resolves this at the scale you need"
- "All candidates fail this axis; recommend drone or field survey instead"

This is **not a failure**; it's a first-class output that prevents silent-wrong answers.

---

## Status Overview

| Story | Name | Stage 0 | Stage 1 Ready | Stage 1 Response | Validation | Notes |
|---|---|---|---|---|---|---|
| 01 | Beavers | ✓ | ✓ | ⏳ | — | Landsat + InSAR combo expected to work |
| 02 | SAR | ✓ | ✓ | ⏳ | — | Latency constraint (< 6 hrs) hardest part |
| 03 | Algal Blooms | ✓ | ✓ | ⏳ | — | Spectral decomposition; Sentinel-2 likely |
| 04 | Permafrost | ✓ | ✓ | ⏳ | — | InSAR stack required; sparse coverage risk |
| 05 | Rusting Rivers | ✓ | ✓ | ⏳ | — | Scale trap; satellite marginal; drone likely needed |
| 06 | Tree Mortality | ✓ | ✓ | ⏳ | — | Phenology confusion trap; Sentinel-2 expected |
| 07 | Glaciohydrology | ✓ | ✓ | ⏳ | — | Firn/ice confusion; MODIS gap-fill strategy |
| 08 | Volcanic Ash | ✓ | ✓ | ⏳ | — | Geostationary + thermal required; hardest latency |
| 09 | Bridge | ✓ | ✓ | ⏳ | — | Scale trap; satellite unlikely; ground sensors instead |
| 10 | LA Olympics | ✓ | ✓ | ⏳ | — | Four sub-needs; multi-axis reasoning test |
| 11 | DC Metro Rail | ✓ | ✓ | ⏳ | — | Scale trap; ground sensors required |

---

## Validation Checklist (Stage 1)

For each story, the Stage 1 response should:

- [ ] Parse the six slots correctly
- [ ] Query earthdata-mcp and return real collections
- [ ] Rank candidates by fitness, not just relevance
- [ ] Surface the "trap for naive picker" from Stage 0
- [ ] Flag datasets in "Wouldn't Work" list with clear reasoning
- [ ] Acknowledge datasets in "Would Work" list
- [ ] Refuse appropriately (don't force-fit if no good match)
- [ ] Recommend data fusion or alternative approaches (e.g., drone, ground sensors, models)
- [ ] Produce structured JSON output for downstream processing

---

## Files of Interest

### For Enrichment Review
- `stories/story-NN.md` — Full enriched story with all six slots, gotchas, suitability criteria
- `stage-0-information-architecture.md` — The template and principles

### For Stage 1 Process
- `stage-1-prompt-template.md` — Detailed instructions for LLM+MCP queries
- `stage-1-prompts/prompt-NN-*.txt` — Instance prompts (one per story)
- `stage-1-responses/response-NN-*.txt` — LLM+MCP responses (to be populated)

### For Overall Status
- `story-index.csv` — Dashboard view across all 11 stories
- `README-stories.md` — This file

---

## Next Actions

**Immediate (Today):**
- [ ] Review story-index.csv; confirm status
- [ ] Spot-check 2–3 enriched stories (stories/story-NN.md) for domain accuracy
- [ ] Run Stage 1 queries for 1–2 pilot stories (start with Beavers, SAR)

**This Week:**
- [ ] Complete Stage 1 runs for all 11 stories
- [ ] Validate Stage 1 responses against Stage 0 predictions
- [ ] Identify quick-win stories (high confidence, clear datasets)
- [ ] Flag hard cases (scale traps, latency, data gaps)

**Downstream (Next Week):**
- [ ] Hand validated stories to Aimee for readiness record authoring
- [ ] Stage 2 exploratory analysis on quick-win datasets
- [ ] Refine Stage 0 enrichments based on Stage 1 learnings

---
