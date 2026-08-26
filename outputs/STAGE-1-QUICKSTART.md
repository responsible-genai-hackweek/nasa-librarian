# Stage 1 Quick Start: Running LLM + earthdata-mcp Queries

You have 11 enriched stories and a prompt template. Here's how to run Stage 1.

## Prerequisites

- Claude Code with access to earthdata-mcp tools (already available)
- A way to run Claude with streaming/multi-turn conversation

## Workflow

### Option A: Run Sequentially (Simple, Conservative)

```bash
# For each story (01 through 11):

1. Read the story:
   cat outputs/stories/story-01-beavers.md

2. Read the prompt:
   cat outputs/stage-1-prompts/prompt-01-beavers.txt

3. Copy prompt into Claude Code with earthdata-mcp access
4. Run the query; wait for LLM+MCP to complete
5. Save response:
   # LLM outputs JSON → copy to outputs/stage-1-responses/response-01-beavers.txt

6. Update story:
   # Edit outputs/stories/story-01-beavers.md
   # Fill in the "Stage 1 Response" section with link to response file
   # Add summary to "Stage 1 Validation" table

7. Update index:
   # Edit outputs/story-index.csv
   # Set story_01: stage_1_response_received = true
   # If validation passes: stage_1_passed = true
```

**Time estimate:** ~10 minutes per story × 11 = ~2 hours total (if earthdata-mcp queries are fast)

### Option B: Run in Batch (Fast, Requires Orchestration)

If you want to parallelize:

1. Generate all 11 prompts at once (already done)
2. Queue them to run in parallel
3. Collect responses
4. Run validation in batch

**Benefit:** All queries run concurrently, total time = ~10 min (slowest query)  
**Cost:** Need orchestration (bash loop, Python script, or manual spreadsheet)

---

## Step-by-Step: Story 01 (Beavers) as Example

### 1. Read the Enriched Story

```bash
cat outputs/stories/story-01-beavers.md
```

**Key sections to internalize:**
- Six Slots (table) — the constraints
- Gotchas & Traps — the failures to avoid
- Would Work — datasets that should appear in the LLM response
- Wouldn't Work — datasets to flag if they show up

### 2. Read the Prompt

```bash
cat outputs/stage-1-prompts/prompt-01-beavers.txt
```

This is the exact text to send to Claude Code + earthdata-mcp.

### 3. Run in Claude Code

```
[Paste the prompt into Claude Code's message box]

Make sure Claude has access to earthdata-mcp tools.
```

The LLM will:
- Parse the six slots
- Query earthdata-mcp with `get_collections()` calls
- Search for Landsat, Sentinel-2, Sentinel-1, regional models
- Validate candidates against the gotchas
- Return structured JSON

### 4. Save the Response

**Expected output:** JSON object with fields:
```json
{
  "story_id": "01",
  "specification_parsed": {...},
  "candidates_found": [
    {"rank": 1, "dataset_name": "...", "confidence": "high", ...},
    ...
  ],
  "recommendation": {...}
}
```

Save this to:
```bash
cat > outputs/stage-1-responses/response-01-beavers.txt << 'EOF'
[Paste the entire JSON response here]
EOF
```

### 5. Update the Story File

Edit `outputs/stories/story-01-beavers.md`:

```markdown
## Stage 1: LLM + earthdata-mcp Query

### Stage 1 Prompt
See: [`../stage-1-prompts/prompt-01-beavers.txt`](../stage-1-prompts/prompt-01-beavers.txt)

### Stage 1 Response
See: [`../stage-1-responses/response-01-beavers.txt`](../stage-1-responses/response-01-beavers.txt)

### Stage 1 Validation

**Did the LLM+MCP picks match the Stage 0 "Would Work" list?**

| Dataset | Stage 0 Prediction | Stage 1 LLM Pick | Rank | Confidence | Match? | Notes |
|---|---|---|---|---|---|---|
| Landsat 8/9 | ✓ Would work | Yes | 1 | High | ✓ | Correctly identified |
| Sentinel-2 | ✓ Would work | Yes | — | High | ✓ | Alternative option |
| Sentinel-1 InSAR | ✓ Would work | Yes | 2 | High | ✓ | For subsidence |
| SMAP | ✗ Won't work | Yes | 3 | **Low** | ✓ | Correctly flagged as naive-picker trap |
| IMERG | ✗ Won't work | No | — | — | ✓ | Not mentioned (good) |

**Gotcha Surfacing:**

Trap: "Temperature" → air temp (wrong) vs. ground temp (right)  
LLM awareness: **✓ Clearly flagged** — noted that "regional permafrost models needed for ground temp; satellite alone insufficient"

**Stage 1 Validation Conclusion:**
✓ PASS — LLM correctly found Landsat and Sentinel-1 as top candidates, properly rejected SMAP with clear reasoning, noted the gap (ground temperature requires model), and recommended data fusion approach.
```

### 6. Update the Index

Edit `outputs/story-index.csv` (find the line for story 01):

```csv
01,Beavers in the Arctic,North Slope Borough,Infectious disease researcher / public health official,stage_1_response_received,true,true,true,true,false,,Strong analysis; Landsat+InSAR+model fusion clear
```

Update fields:
- `status`: "stage_1_response_received"
- `stage_1_response_received`: true
- `stage_1_passed`: true (if validation passed) or false (if validation failed)
- `blockers`: "" (empty if none) or specific issue

---

## Success Criteria

A Stage 1 response passes validation if:

✓ **Parsed constraints correctly**
- Unit of analysis: stated in the response
- Resolution requirement: mentioned
- Time window: acknowledged

✓ **Found real datasets**
- Landsat collections returned? (if in "Would Work")
- Sentinel-2 collections returned? (if in "Would Work")
- Other relevant collections?

✓ **Ranked by fitness, not just relevance**
- Landsat ranked #1–2 (not buried)
- SMAP ranked low if returned (not top-5)
- Reasoning mentions resolution, time span, latency, not just keyword match

✓ **Surfaced the gotchas**
- Mentioned the "trap for naive picker"
- Noted regional caveats
- Flagged scale mismatches

✓ **Produced structured output**
- JSON parseable
- Confidence scores
- Reasoning for each candidate

---

## Common Pitfalls to Avoid

### Pitfall 1: LLM Ranks SMAP High
**Problem:** SMAP appears in top-3 candidates despite being listed in "Wouldn't Work"  
**Fix:** Explicitly tell LLM in prompt: "This dataset is explicitly listed in 'Wouldn't Work'; if you find it, flag it as naive-picker trap, rank low, and explain why."

### Pitfall 2: LLM Misses earthdata-mcp Collections
**Problem:** Response says "No Sentinel-2 collections found"  
**Fix:** Ensure earthdata-mcp tools are available and try different search queries (e.g., "Sentinel-2 Level 2A", "S2A", "Copernicus")

### Pitfall 3: Response Doesn't Mention Refusal
**Problem:** Story says "At least one question is unanswerable at the scale asked" but LLM found a dataset anyway  
**Fix:** Explicitly design Stage 2 validation to test. If LLM didn't refuse, that's OK — Stage 2 will prove whether the dataset actually works.

### Pitfall 4: Confidence Is Too High Everywhere
**Problem:** All candidates marked "high confidence"  
**Fix:** Remind LLM: "Medium confidence if fits most specs but has a gotcha. Low confidence if mentioned in 'Wouldn't Work'."

---

## After All 11 Stories Complete

Once you have responses for stories 01–11:

1. **Aggregate results:**
   ```bash
   # How many stories passed Stage 1 validation?
   grep "stage_1_passed: true" outputs/story-index.csv | wc -l
   
   # Which stories have blockers?
   grep -v "^blockers" outputs/story-index.csv | grep -v ",$" | awk -F, '{print $1 ": " $NF}'
   ```

2. **Identify patterns:**
   - Which stories had clear dataset matches?
   - Which had refusals (appropriate or not)?
   - Which had data gaps (e.g., ground temperature, sub-10m resolution)?

3. **Plan Stage 2:**
   - Stories that passed → ready for exploratory data analysis
   - Stories that failed → do we refine the spec, or refusal is correct?

4. **Hand to Aimee:**
   - "Here are the 3–5 datasets we'll focus on"
   - "These are the Stage 1 validation results; they match our predictions"
   - → Aimee hand-authors readiness records

---

## Running It: Your Choice

**Simple:** Copy prompt into Claude Code one-by-one, save responses.  
**Faster:** Write a bash loop or Python script to parallelize.  
**Ideal:** Use Claude Code's API programmatically and collect all responses at once.

Pick whatever works for your workflow. The structure supports all three.

---

## Questions?

If a Stage 1 response is confusing or seems wrong:

1. **Check the prompt:** Is it clear? Does it have the right constraints?
2. **Check the gotchas:** Are they obvious enough for the LLM to catch?
3. **Check the earthdata-mcp response:** Did the tool return the expected collections?
4. **Re-read the Stage 0 spec:** Did we enrich the story correctly?

If still stuck, update the story file with a note ("Stage 1 blocker: SMAP returned as #1 despite being in 'Won't Work'; investigate why") and move on. Stage 2 will clarify.

---
