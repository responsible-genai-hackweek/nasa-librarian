# SlideRule — J.P. Swinski & Carlos Ugarte

**Talk notes + research · 2026-08-25 · Responsible Gen-AI for NASA Earthdata, Seattle**

J.P. Swinski reports on an MCP for **SlideRule**, built with Carlos Ugarte. SlideRule is
an open-source, on-demand science data processing service — server-side processing of
ICESat-2, GEDI, ArcticDEM/REMA and HLS in AWS, reading NASA HDF5 straight from S3.
Authors include David Shean, Swinski, Ben Smith, Tyler Sutterley, Scott Henderson,
Ugarte, Eric Lidwa and Thomas Neumann (NASA GSFC + UW).

## The pattern, confirmed a fourth time

Every system owner is wrapping their service in MCP: Earthmover (Arraylake), NASA
(`earthdata-mcp`), Rajat's orbit (`geocr_mcp`), now SlideRule. Joe's slide 27 named it —
*the provider ships the workflow, not just the API.*

**SlideRule occupies a different box from the others.** CMR finds, earthaccess fetches;
SlideRule *computes* — you hand it a region and parameters, it runs the algorithm
server-side and returns results. It is downstream of the choice, and it collapses fetch
and process into one call.

## What they built around the MCP — and why it matters to us

This is the mature instance. Look at what the org has shipped in the last three weeks:

### 1. `sliderule-schema-server` — the three-tier production model

A JSON-only CloudFront distribution serving schema endpoints at stable URLs. The
architecture is the finding:

```
authored/    (human-edited)  ─┐
                              ├─▶  merged/  ─▶ S3 ─▶ https://schema.../schema/icesat2.json
generated/   (tool-emitted)  ─┘
```

`merged/` is **committed to git so reviewers see the S3-bound diff on every PR**, and
`make verify` asserts it still matches what the merge script would produce today.

**That is the readiness record's production model, already designed and running.** Our
record has exactly this split: `geocr:spatialResolution` can be tool-generated from
STAC; `min_meaningful_area_km2`, `quality_flag_convention` and `cautions[]` must be
human-authored. Adopt the pattern wholesale — authored + generated → merged, versioned,
reviewable, served at a stable URL.

### 2. `sliderule-search-server` — documentation behind retrieval, not in context

`POST /docsearch/search` over the SlideRule docs corpus: cosine similarity on
precomputed chunk embeddings fused with IDF-weighted lexical overlap via reciprocal rank
fusion. Consumed by a `sliderule-docsearch` skill.

Note what they did **not** do: they did not stuff documentation into tool descriptions or
a system prompt. They put it behind a retrieval endpoint. That is the answer to the
context-budget problem Joe and Jason both raised, and the contrast with NASA's
4,812-token `instructions` block is stark.

### 3. `open-agent-skills-test-harness` — our gold set, already built

A cross-agent eval harness: one eval, run against Claude Code, Codex, GitHub Copilot and
AntiGravity, graded by **deterministic assertions plus an LLM judge** on a rubric. Evals
live per-skill; scenarios combine skills with a pinned runner and model. The README calls
the harness "the durable purpose of the repo" and it takes external skills via
`--skills-root`.

**This is directly reusable for D5 and the gold set.** We need refusal-accuracy scoring
across a fifteen-question set; this is that machinery, cross-platform, from a team in the
building.

## Does it close our gap? No — and the reason is precise

SlideRule's schema server describes **their API**: request parameters, and the output
columns each endpoint returns. That is *interface* knowledge — how to call the service
correctly. It is not *fitness* knowledge: nothing in it says whether ICESat-2 can answer
a question about a farmer's field.

**Same architecture, different content.** They have proven the pattern works and shipped
the scaffolding. The fitness content remains unfilled — which strengthens our position
rather than weakening it. We are not inventing an odd new thing; we are applying a
pattern four teams have independently converged on, to the one content type nobody has
filled.

## The axis SlideRule exposes that our table was missing

Our persona set is all gridded — HLS, SMAP, IMERG, MOD16A2 — which hid a whole axis.

ICESat-2 is a **profiling lidar, not an imager.** Its along-track sampling is fine, but
beam pairs are kilometres apart and the repeat cycle is ~91 days. GEDI is a sampling
lidar from the ISS: discrete footprints, non-repeating, and only within its latitude
band. **A parcel may simply never have been observed.**

So a resolution number is not enough. "17 m along-track" sounds superb and tells you
nothing about whether the instrument ever crossed the field in question.

**New axis — sampling geometry:** *is this a complete field, or a sample of one?* And
the operand for it was already in the record and unused: the original memo's shape block
carries `structure gridded | swath | point`. The field existed; the comparison did not.

## Actions

- Adopt the **authored + generated → merged** production model for readiness records.
- Evaluate `open-agent-skills-test-harness` for the gold set before building our own.
- Add **sampling geometry** to the fitness axes; extend `structure` to include `track`
  and `footprint`.
- Ask J.P. and Carlos: did they consider serving fitness/suitability alongside the
  schema, and did the retrieval endpoint measurably improve tool-call correctness?

---

## J.P.'s three questions — a classification scheme for agentic workflows

He poses three, and they shape the problem domain:

1. **Is the executor an agent or a human?**
2. **Is the targeted output code or answers?**
3. **Is the goal to reduce cost or to improve robustness?**

### Where we sit

| Question | Our answer | Why |
|---|---|---|
| Executor | **Human** | D6 already leans this way. The notebook keeps a human in the loop *by construction*, not by prompt instruction. |
| Output | **A decision with its justification, then code** | We never compute whether the county is drying out. We answer the *meta-question*: what could answer yours, and what couldn't. |
| Goal | **Robustness** | The failure we target is silent. Cost is what every incumbent already optimises, and they are good at it. |

### The answers are not independent

That is the insight worth keeping. **Choosing robustness largely forces the other two.**

- If the *agent* executes, nothing is reviewed, and the judgement that matters most —
  the fitness call — becomes unauditable. Agent execution undermines robustness.
- If the output is *answers* rather than code plus reasoning, the human cannot check the
  work, and over-trust is the default. Answers undermine robustness.

So `{human executor, code + justification, robustness}` is a **coherent cluster**, and
mixing across clusters produces incoherent systems. The common "AI for science" demo
sits in the opposite cell — agent executes, emits answers, optimises cost — which is
exactly the configuration that maximises silent error.

### Consequence 1 — we should expect to look worse on speed

If the goal is robustness, the system must be **willing to cost more**. An interview
costs the user time. A refusal costs them their preferred answer. Both are the product
working correctly. Someone will benchmark this and "it asked me two questions" will read
as a defect unless we frame it first.

The reconciliation is the disagreement rule: we ask only what would change the answer,
so we are robust without being gratuitously slow. Two questions, not ten.

This also settles the metric argument in **D5** — refusal accuracy, never time-to-first-
dataset.

### Consequence 2 — the compatibility report is the universal output; the notebook is conditional

A real clarification of the architecture. The memo's flow ends at a co-registered cube,
which makes refusal look like a degenerate case where the pipeline fails to produce its
output.

Invert it. **The report is always produced. The notebook is produced only on a positive
verdict.** For the snow observer there is no notebook at all — the deliverable is the
refusal and what the closest available data cannot show. That makes refusal a
first-class output rather than an error path, which is what D5 says we are scoring.

### Consequence 3 — the executor is constant; the reviewed artifact varies by persona

Separate *who runs it* from *what the human checks*.

| Persona | Executes | Actually reviews |
|---|---|---|
| Researcher | the notebook | the notebook and the selection record — it goes in the methods section |
| Land manager | the notebook | the **compatibility report**; they may not read Python at all |
| Snow observer | nothing | the refusal and its reasoning |

Human execution is about accountability, not about everyone reading code.
