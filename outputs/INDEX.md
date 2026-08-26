# User Stories Pipeline: Complete Index

## Quick Navigation

**New here?** Start with: [`README-stories.md`](README-stories.md)

**Want to understand the versioning system?** Read: [`DIRECTORY-STRUCTURE.md`](DIRECTORY-STRUCTURE.md) and [`VERSIONING.md`](VERSIONING.md)

**Ready to run Stage 1?** Go to: [`STAGE-1-QUICKSTART.md`](STAGE-1-QUICKSTART.md)

**Dashboard (all 11 stories):** [`story-index.csv`](story-index.csv)

---

## Files Overview

### Reference Documents (Read First)

| File | Purpose |
|---|---|
| **README-stories.md** | Overview of the pipeline, directory structure, status dashboard |
| **STRUCTURE.md** | Why we designed it this way; how to extend it |
| **STAGE-1-QUICKSTART.md** | Step-by-step guide to running Stage 1 queries |
| **stage-0-information-architecture.md** | Template and principles for enrichment |
| **stage-1-prompt-template.md** | Detailed instructions for LLM+MCP queries |

### Tracking & Status

| File | Purpose |
|---|---|
| **story-index.csv** | Machine-readable dashboard; query with grep/awk |

### Individual Stories (One Per Story)

| File | Story |
|---|---|
| **stories/story-01-beavers.md** | Beavers in the Arctic (infectious disease + permafrost) |
| **stories/story-02-sar.md** | North Slope Borough Search & Rescue |
| **stories/story-03-algal-blooms.md** | Algal blooms & fjord productivity |
| **stories/story-04-permafrost.md** | Permafrost subsidence risk |
| **stories/story-05-rusting-rivers.md** | Iron oxidation in mining-affected streams |
| **stories/story-06-tree-mortality.md** | Tree mortality detection |
| **stories/story-07-glaciohydrology.md** | Hydropower + glacier meltwater |
| **stories/story-08-volcanic-ash.md** | Volcanic ash for air traffic control |
| **stories/story-09-bridge.md** | Bridge structural monitoring |
| **stories/story-10-olympics.md** | LA Olympics 2028 multi-hazard |
| **stories/story-11-dc-metro.md** | DC Metro rail thermal expansion |

*Each story file contains:*
- Raw story (from issue #3)
- Stage 0 enrichment (six slots, gotchas, suitability criteria)
- Links to Stage 1 prompt and response
- Stage 1 validation table
- Placeholder for Stage 2 results

### Stage 1 Prompts (Input to LLM+MCP)

One prompt per story. Ready to use.

| File | Story |
|---|---|
| **stage-1-prompts/prompt-01-beavers.txt** | Story 01 |
| **stage-1-prompts/prompt-02-sar.txt** | Story 02 |
| *(etc., 11 total)* | *(continue)* |

*Each prompt contains:*
- Enriched story context
- Six slots (explicit constraints)
- Gotchas & traps
- Instructions for LLM
- Expected output format (JSON)
- Success criteria

### Stage 1 Responses (Output from LLM+MCP)

To be populated after running queries.

| File | Story |
|---|---|
| **stage-1-responses/response-01-beavers.txt** | Story 01 (empty until run) |
| **stage-1-responses/response-02-sar.txt** | Story 02 (empty until run) |
| *(etc., 11 total)* | *(continue)* |

*Each response will contain:*
- Parsed constraints
- Candidate datasets from earthdata-mcp
- Fitness ranking (not relevance ranking)
- Gotcha analysis for each candidate
- Recommendation
- Refusal (if appropriate)

---

## The Pipeline at a Glance

```
Raw Stories (issue #3)
        ↓
[Stage 0: Enrichment — COMPLETE]
        ↓
stage-0-information-architecture.md (11 stories enriched)
        ↓
stories/story-NN.md (one file per story)
        ↓
[Stage 1: LLM + earthdata-mcp — IN PROGRESS]
        ↓
stage-1-prompts/prompt-NN.txt (ready to use)
        ↓
[Run Claude Code with earthdata-mcp]
        ↓
stage-1-responses/response-NN.txt (to be populated)
        ↓
stories/story-NN.md → Update validation table
        ↓
story-index.csv → Update status
        ↓
[Stage 2: Exploratory Data Analysis — FUTURE]
        ↓
Test top datasets; hand-author readiness records
```

---

## Status Summary

| Stage | Status | Deliverables |
|---|---|---|
| **Stage 0: Enrichment** | ✓ Complete | 11 enriched stories + reference architecture |
| **Stage 1: LLM+MCP Query** | ⏳ Ready | 11 prompts ready; responses to be generated |
| **Stage 2: Data Exploration** | 📋 Planned | (Depends on Stage 1 results) |

---

## Getting Started

### For Project Leads
1. Open `story-index.csv` — see status across all 11
2. Read `README-stories.md` — understand the process
3. Pick 2–3 pilot stories for Stage 1 (suggest: Beavers, SAR, Algal Blooms)

### For Data Scientists / Domain Experts
1. Open `stories/story-NN.md` for the story you own
2. Review Stage 0 enrichment (six slots, gotchas)
3. Correct or add domain knowledge
4. Provide to LLM+MCP team for Stage 1

### For Engineers / LLM Operators
1. Read `STAGE-1-QUICKSTART.md`
2. Pick a pilot story (e.g., Story 01: Beavers)
3. Follow the step-by-step guide
4. Run Stage 1 query; save response; validate

### For Future Readers
1. Start with `README-stories.md` (overview)
2. Read `STRUCTURE.md` (why it's organized this way)
3. Pick a story and read its markdown file (e.g., `stories/story-01-beavers.md`)
4. Follow the lifecycle from raw → enriched → queried → validated

---

## Key Concepts

### Six Slots
Every story must specify:
1. **Unit of Analysis** (what scale?)
2. **Time Window** (how many years?)
3. **Cadence** (how often?)
4. **Variable Meaning** (what to measure?)
5. **Effect Size** (what change matters?)
6. **Latency** (how fast?)

These become queries to earthdata-mcp.

### Gotchas & Traps
Each story identifies where naive satellite-data users fail:
- Confusing air temp with ground temp
- Picking datasets that are too coarse
- Mistaking relevance for fitness

Stage 1's job is to surface these **before** the user wastes time.

### Refusal as Success
Sometimes the right answer is "nothing fits." This is not a failure; it's a first-class output that prevents silent-wrong answers.

---

## Metrics & Validation

**Stage 1 Success Criteria:**
- ✓ Found datasets in "Would Work" list
- ✓ Ranked them high (top 3)
- ✓ Flagged datasets in "Wouldn't Work" as low confidence + explained why
- ✓ Surfaced the "trap for naive picker"
- ✓ Refused appropriately (if no good match)

**Stage 2 Success Criteria:**
- ✓ Retrieved and tested top datasets
- ✓ Confirmed Stage 1 prediction matched reality
- ✓ Authored readiness records

---

## Questions?

- **Why structured this way?** → Read `STRUCTURE.md`
- **How do I run Stage 1?** → Read `STAGE-1-QUICKSTART.md`
- **What's the overall process?** → Read `README-stories.md`
- **What does enrichment look like?** → Read `stories/story-01-beavers.md`
- **How do I query earthdata-mcp?** → Read `stage-1-prompt-template.md`

---

**Last updated:** 2026-08-26  
**Pipeline status:** Stage 0 complete; Stage 1 ready; Stage 2 pending
