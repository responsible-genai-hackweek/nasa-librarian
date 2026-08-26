# Story Pipeline Structure: Why This Design

## The Problem We Solved

You had 11 raw user stories from issue #3. Converting them to validated dataset queries requires:

1. **Enriching** raw tacit knowledge → explicit six-slot specification
2. **Querying** earthdata-mcp with that specification
3. **Validating** results against Stage 0 predictions
4. **Tracking** what changed at each stage

A linear markdown chain (stage-0.md → stage-1.md → stage-2.md) made tracing backward impossible. Where did the Beavers story end up? It's scattered across 4 files.

## The Solution

**One file per story + indexed CSV + separate prompt/response directories.**

### File Hierarchy

```
outputs/
├── story-index.csv                    # [MACHINE READABLE] Dashboard across all 11
├── README-stories.md                  # [HUMAN READABLE] Process overview
├── STRUCTURE.md                       # [THIS FILE] Design rationale
├── stage-0-information-architecture.md # [REFERENCE] Template and principles
├── stage-1-prompt-template.md         # [REFERENCE] Instructions for LLM
│
├── stories/
│   ├── story-01-beavers.md           # [HUMAN EDITABLE] Full lifecycle of Story 01
│   ├── story-02-sar.md               # [HUMAN EDITABLE] Full lifecycle of Story 02
│   └── ... (11 total)
│
├── stage-1-prompts/
│   ├── prompt-01-beavers.txt         # [MACHINE INPUT] LLM prompt for Story 01
│   ├── prompt-02-sar.txt             # [MACHINE INPUT] LLM prompt for Story 02
│   └── ... (11 total)
│
└── stage-1-responses/
    ├── response-01-beavers.txt       # [MACHINE OUTPUT] LLM response for Story 01
    ├── response-02-sar.txt           # [MACHINE OUTPUT] LLM response for Story 02
    └── ... (11 total)
```

### Design Decisions

#### 1. One Story File Per Story (stories/story-NN.md)

**Benefit:** Audit trail is in one place.
```markdown
---
story_id: "01"
status: "stage_1_response_received"
---

# Story 01: Beavers

## Raw Story (From Issue #3)
[original text]

## Stage 0: Enriched Story
[enrichment from stage-0-issue-body.md or authored directly]

### Six Slots
[table]

### Gotchas & Traps
[bullet points]

### Would Work / Wouldn't Work
[list]

## Stage 1: LLM + earthdata-mcp

### Stage 1 Prompt
See: [`../stage-1-prompts/prompt-01-beavers.txt`](../stage-1-prompts/prompt-01-beavers.txt)

### Stage 1 Response
See: [`../stage-1-responses/response-01-beavers.txt`](../stage-1-responses/response-01-beavers.txt)

### Stage 1 Validation
Did LLM picks match predictions? Table comparing Stage 0 "Would Work" vs Stage 1 picks.

## Stage 2 [Future]
[placeholder for data exploration results]
```

**Why separate files for prompts/responses?**
- Prompts are long (500+ tokens) — keep story markdown clean
- Responses are long (LLM output) — don't clutter the story
- Both are version-tracked; git history matters
- Story markdown links to them, so the narrative is complete even if you don't read the full prompt

#### 2. Indexed CSV (story-index.csv)

**Benefit:** Machine-queryable dashboard.

```csv
story_id,name,status,stage_0_complete,stage_1_response_received,stage_1_passed,blockers
01,Beavers,stage_1_response_received,true,true,false,
02,SAR,stage_1_prompt_ready,true,false,,Clarify latency tolerance
03,Algal Blooms,enriched,true,false,,
```

Can ask:
```bash
# How many stories are ready for Stage 1?
grep "stage_1_ready: true" outputs/story-index.csv | wc -l

# Which have blockers?
grep -v "^blockers" outputs/story-index.csv | grep -v ",," | awk -F, '{print $1, $NF}'
```

#### 3. Separate Prompt & Response Directories

**Benefit:** Clear input/output separation for automation.

- `stage-1-prompts/` = machine input (instructions to LLM)
- `stage-1-responses/` = machine output (LLM results)
- `stories/` = human narrative (links both)

When you build a validation script, it can:
```python
for story_id in range(1, 12):
    prompt = read(f"stage-1-prompts/prompt-{story_id:02d}-*.txt")
    response = read(f"stage-1-responses/response-{story_id:02d}-*.txt")
    result = validate(prompt, response, spec)
    update_csv(story_id, result)
```

#### 4. YAML Frontmatter in Story Markdown

**Benefit:** Metadata is machine-readable without parsing markdown.

```yaml
---
story_id: "01"
name: "Beavers in the Arctic"
status: "stage_1_response_received"
stage_0_complete: true
stage_1_response_received: true
stage_1_passed: false
blockers: ""
---
```

Can extract with:
```bash
grep -h "^story_id:" outputs/stories/*.md | sort
```

---

## Workflow

### Week 1: Enrichment (Complete)

```
Raw stories (issue #3)
       ↓
Enrichment (Claude, domain experts review)
       ↓
stories/story-NN.md (with Stage 0 complete)
       ↓
stage-0-issue-body.md (centralized reference)
```

**Deliverable:** 11 enriched story files + one reference document

### Week 2: Stage 1 Queries

```
stories/story-NN.md (read Six Slots + Gotchas)
       ↓
stage-1-prompt-template.md (instantiate for each story)
       ↓
stage-1-prompts/prompt-NN.txt (write once, use many times)
       ↓
LLM + earthdata-mcp (query)
       ↓
stage-1-responses/response-NN.txt (save raw output)
       ↓
stories/story-NN.md (link response, fill validation table)
       ↓
story-index.csv (update status)
```

**Deliverable:** 11 responses + validation tables + updated index

### Week 3: Validation & Refine

```
For each story:
  - Does Stage 1 response match Stage 0 "Would Work" list?
  - Did LLM surface gotchas?
  - Are there data gaps?
  - Is a refusal warranted?

stories/story-NN.md (update validation table)
story-index.csv (mark as "stage_1_passed" or "stage_1_refusal")

Aggregate: Which stories pass? Which fail? Why?
```

**Deliverable:** Validation summary + refined story files

### Week 4: Stage 2 (Data Exploration)

```
stories/story-NN.md (Stage 1 validation ✓)
       ↓
Retrieve top-ranked datasets
       ↓
Pilot analysis (NDVI time series, InSAR stack, etc.)
       ↓
stories/story-NN.md (Section "Stage 2: Exploratory Results")
       ↓
Hand to Aimee: Use validated stories to author readiness records
```

---

## Benefits of This Structure

### For Humans
- **Narrative flow:** Read `stories/story-01-beavers.md` → see full lifecycle from raw → enriched → queried → validated
- **Traceability:** Every decision is timestamped and documented
- **Edit safety:** Change one story without affecting others

### For Machines
- **CSV is queryable:** `grep`, `awk`, Python can parse status
- **Prompts and responses are separate files:** Easy to version-track, diff, or re-run
- **YAML frontmatter:** Can extract metadata without markdown parsing
- **Linked structure:** Automation can follow links (story → prompt → response)

### For the Project
- **Audit trail:** Why did this story's dataset choice change? Git history shows the exact commit
- **Parallelization:** Each story can be worked on independently
- **Incremental progress:** Don't need all 11 stories done before validating the first 3
- **Future extension:** Adding Stage 2, Stage 3 is just adding sections to story markdown

---

## Example: Tracing a Single Story

**Question:** Why did Story 01 (Beavers) end up with IMERG as a candidate?

**Trace:**

1. **Did Stage 0 predict this?**
   ```bash
   grep -A 5 "Wouldn't Work" outputs/stories/story-01-beavers.md
   # Output: "SMAP (36 km): Too coarse", "IMERG (11 km): Too coarse"
   ```
   → Stage 0 says "shouldn't work"

2. **What prompt was used?**
   ```bash
   cat outputs/stage-1-prompts/prompt-01-beavers.txt | head -50
   # Shows that gotcha was highlighted: "Too coarse"
   ```

3. **What did LLM actually do?**
   ```bash
   cat outputs/stage-1-responses/response-01-beavers.txt | grep -i "imerg" -A 5
   # Shows LLM's reasoning for including IMERG
   ```

4. **Was it a mistake?**
   ```bash
   grep -A 2 "confidence.*low" outputs/stage-1-responses/response-01-beavers.txt | grep -i "imerg"
   # If "low confidence" and "warning: naive trap", then LLM correctly flagged it
   # If "high confidence", then LLM failed the validation
   ```

5. **Update the story:**
   ```bash
   # Edit stories/story-01-beavers.md, fill Stage 1 Validation table
   # Update story-index.csv: stage_1_passed = false, add blocker note
   ```

**Result:** Git history shows exactly what changed, when, and why.

---

## Testing the Structure

### Quick Validation

```bash
# Are all story files well-formed YAML?
for f in outputs/stories/*.md; do
  head -5 "$f" | grep -q "^---$" && echo "$f: ✓" || echo "$f: MISSING YAML"
done

# Do all stories have stage-1-prompts?
for id in 01 02 03 04 05 06 07 08 09 10 11; do
  [ -f "outputs/stage-1-prompts/prompt-$id"*.txt ] && echo "Story $id: ✓" || echo "Story $id: MISSING"
done

# Are story IDs in CSV consistent with file names?
grep "^story_id:" outputs/stories/*.md | awk '{print $3}' | sort -u > /tmp/story_ids.txt
awk -F, 'NR>1 {print $1}' outputs/story-index.csv | sort -u > /tmp/csv_ids.txt
diff /tmp/story_ids.txt /tmp/csv_ids.txt
```

### Full Workflow Test

Once Stage 1 responses are in:

```bash
# Count completed stages per story
awk -F, 'NR>1 {
  stage0 = ($4 == "true") ? "S0✓" : "S0✗"
  stage1 = ($6 == "true") ? "S1✓" : "S1✗"
  stage1_resp = ($7 == "true") ? "R✓" : "R✗"
  print $1 ": " stage0 " " stage1 " " stage1_resp " - " $2
}' outputs/story-index.csv

# Identify blockers
awk -F, 'NR>1 && $9 != "" {print $1 ": BLOCKER - " $9}' outputs/story-index.csv
```

---

## Extending to Future Stages

When you're ready for Stage 2 (data exploration), the structure adapts seamlessly:

```markdown
## Stage 2: Exploratory Data Analysis

### Stage 2 Plan
[what we'll test]

### Stage 2 Results
[what we found]

### Stage 2 Validation
[did it work?]
```

Just add sections to `stories/story-NN.md`. The narrative flow remains intact, git history tracks changes, and CSV gets updated with progress.

---
