# Progress Log

## [2026-08-26] (Updated)

### Additional Work: Story Pipeline Structure

After completing Stage 0 enrichment and reflecting on how to track the evolution of stories through refinement → LLM query → validation → data exploration, designed and implemented a structured file system:

**Deliverables:**
- `outputs/story-index.csv` — Machine-readable dashboard across all 11 stories
- `outputs/stories/story-NN.md` — One file per story with full lifecycle (raw → enriched → queried → validated)
- `outputs/stage-1-prompts/prompt-NN.txt` — Instance prompts for LLM+MCP queries
- `outputs/stage-1-responses/response-NN.txt` — Response outputs (to be populated)
- `outputs/README-stories.md` — Process overview and user guide
- `outputs/STRUCTURE.md` — Design rationale and architecture decisions
- `outputs/stage-1-prompt-template.md` — Detailed instructions for LLM (how to query earthdata-mcp, validate candidates, surface gotchas)
- `outputs/stage-0-information-architecture.md` — Enrichment template (persisted from prior work)

**Example:** `outputs/stories/story-01-beavers.md` contains:
- Raw story (from issue #3)
- Stage 0 enrichment (six slots, gotchas, suitability criteria)
- Links to Stage 1 prompt and response files
- Validation table (when Stage 1 completes)
- Placeholder for Stage 2 results

**Why this structure?**
- One file per story = clean audit trail
- CSV index = machine-queryable dashboard
- Separate prompt/response dirs = clear input/output for automation
- YAML frontmatter = metadata extraction without markdown parsing
- Linked narrative = humans can follow the story lifecycle; machines can parallelize

**Next action:** Run Stage 1 LLM+MCP queries on all 11 stories. Populate `stage-1-responses/`. Validate against Stage 0 predictions.

---

## [2026-08-26]
### What I did
- Ran live experiments with Claude Code, JupyterAI, and earthdata-mcp in the CryoCloud
  2i2c-JupyterHub. See [notes/2026-08-26-cryocloud-experiments.md](notes/2026-08-26-cryocloud-experiments.md).
- Tested LLM + MCP discovery and filtering on a domain question. Successfully narrowed
  broad search results (287 collections) to a focused set closer to the actual need.

### What I learned
- **The operator (agent + MCP) is solved.** Existing components (earthdata-mcp,
  Claude Code, JupyterAI, SlideRule's schema server, GeoCroissant, ChatGSFC, Element
  84's geospatial agent) already interlock into a working "librarian." Working alone,
  none of us would have known about all these recently-built pieces.
- **The hackweek created a discovery condition** — seeing all components in one room,
  learning what each team built, understanding how they interlock. This is the
  efficiency gain: not a new technology, but visibility into the existing landscape.
- **The real gap is the operand, not the operator.** The agent can search and filter.
  What it cannot do — without the readiness record — is *refuse*, *compare*, or
  *co-register*. Those require dataset-side facts (resolution, uncertainty, quality
  flags, cautions). The record is cheap to hand-author and unlocks judgment.
- **This reframes what the week delivers.** Not a new agent, but the characterization
  layer that makes existing agents capable of judgment rather than just good at search.
  Betting on the record (the operand) rather than the agent (the operator) is now
  validated by experiment.

### Next steps
- [ ] Assign an owner for D1 (region, question, persona)
- [ ] Hand-author five Dataset Readiness Records — this is now the priority
- [ ] The desk: implement the disagreement-driven interview loop
- [ ] One access recipe that runs

### Blockers
- D1 still unassigned

---

## [2026-08-25]
### What I did
- Read the 2026-08-24 architecture memo and reconciled `docs/` against it —
  `objective.md` rewritten from template placeholder, `decisions.md` and
  `progress.md` restored with real content.
- Attended Joe Hamman (Earthmover) on MCP; notes in
  [notes/2026-08-25-joe-hamman-mcp.md](notes/2026-08-25-joe-hamman-mcp.md).
- Researched the Earthdata MCP landscape and probed the live NASA server. **D3
  settled** — the librarian is a client of `nasa/earthdata-mcp`, not an integration.
- **R1 answered for the load-bearing field.** Resolution is partly in CMR, as free
  text in mixed units, and null for the one dataset that matters most.
- Rewrote the gap statement in `objective.md` around a single verified case, and
  re-baselined the evaluation tracks against the MCP-equipped agent.

### What I learned
- The repo was a scaffold only: `src/`, `data/` and `outputs/` hold nothing but
  README/gitkeep files. All project thinking so far lives in the memo artifact, not
  in git.
- The shared link on the artifact serves collaborators a **pinned earlier version**,
  not the current one. Anyone working from that link may be a revision behind.
- **The gap has a crisp statement.** The catalog describes what a dataset *is*;
  fitness is a relation between a dataset and a question; nothing holds the
  dataset-side facts needed to compute it. The record is the operand, the librarian
  the operator — and the operand is missing, which is why it is the week's priority.
- **Availability-refusal and fitness-refusal are different problems.** NASA solved the
  first. Conflating them means claiming credit for finished work.
- **The claim is fitness blindness, not popularity bias.** With an MCP in the loop the
  agent queries the catalog rather than its pretraining, so the popularity argument no
  longer describes the failure we can demonstrate. Sharper claim, better defended.
- **Keenan's layer overlaps the incumbent most.** NASA's server already emits
  `earthaccess` snippets with calibration parameters as comments. What is genuinely
  absent is CI verification, recipes as collection assets, and co-registration.
- **Scope absorption is the strategic risk.** That server was created in Nov 2025 and
  is actively growing to absorb exactly this kind of guidance. Bet on artifacts a
  system prompt cannot hold — the record schema and the STAC convention (D7).

### Next steps
- [ ] Assign an owner for D1 (region, question, persona) — it gates fixtures, gold
      set and demo script
- [ ] Test R1 on three datasets: can fitness knowledge be extracted from DAAC docs,
      or must it be hand-curated?
- [ ] Hand-write three Dataset Readiness Records so the librarian has something to
      develop against
- [ ] Write the readiness record JSON Schema, with `{value, source, confidence}`
      wherever a value exists

### Blockers
- D1 unassigned — fixtures, gold set and demo script can't start without it.

---

## [2026-08-24]
### What I did
- Workshop day at UW eScience Institute (Responsible Generative AI). Converged three
  separate proposals into one architecture with Keenan, Aimee, Sarah, Julia and Rajat.
- Drafted the architecture memo:
  https://claude.ai/code/artifact/37a5c8c4-ee37-4c90-a97b-f9cdaa7f9dd5
- Divided the work: technical services / readiness assessment (Aimee), reference desk
  / librarian agent (James), circulation / access recipe (Keenan).

### What I learned
- **The missing row.** NASA's *Which data tool is right for you?* (v2, July 2025)
  scores ten tools across nineteen rows, every one asking what the tool can do. None
  asks whether the data is fit for the asking. That absent row is the project.
- **The chart hands us three things** in the agency's own voice: the premise, a schema
  someone already designed, and an evaluation baseline the librarian can be scored
  against. Its tri-state encoding is worth borrowing for the compatibility report —
  the audience already reads it.
- **Only 1 of 10 tools is a clean "no programming skills required"** (CMR Search API
  is the yes, VEDA Dashboard partial). The tools that reach non-experts are the ones
  that ask least of them.
- **The shared record is the project.** Three skills spanning production, selection
  and use — fitness knowledge produced upstream and carried downstream, where today it
  lives in PDFs and in the heads of people who have used a product for a decade. That
  chain is the responsible-AI claim, and it's stronger than any one agent being clever.
- **Two skills, one record — never two records.** Reconciling documents at query time
  is work the librarian should never do.
- **Catalog beats documentation as a discoverability channel.** The open-web channel is
  popularity-biased and frozen at training cutoff; the catalog is current by
  construction and conformance is binary. The catalog is the fix a small DAAC can
  afford, and the one that stays true after the next model is trained.
- **CI is what makes a recipe trustworthy** rather than decorative. A recipe that
  passed an hour ago is a fact; a recipe in a PDF is a hope.
- **The interview and the qualified refusal are the contribution** — ordinary behaviour
  for a librarian, almost unknown in recommendation systems.

### Next steps
- [ ] Settle the seven open decisions — see [decisions.md](decisions.md)
- [ ] Test R1 (can fitness knowledge be extracted from DAAC docs?)

### Blockers
- None recorded.
