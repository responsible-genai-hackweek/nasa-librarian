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
