# Decision Log

## Open decisions

Ten decisions and two open risks. D1–D7 and R1 carry over from the architecture memo
of 2026-08-24; D8, D9, D10 and R2 opened 2026-08-25. Ordered by when they stop being free to
answer — the first three are gated on nothing and block everything.

Status values: `open` · `leaning` · `decided` · `superseded`.
When one is settled, move it down to **Decided** below with context and consequences.

### At a glance

| # | Question | Status | Needs |
|---|---|---|---|
| **D1** | Which region, question and persona? | **OPEN** | **an owner, today** |
| D2 | Which providers may the librarian search? | leaning — three | confirm |
| D3 | Existing Earthdata MCP server, or reimplement? | **settled** — be a client | confirm with Aimee |
| D4 | Which role goes on stage? | leaning — open with the refusal | follows D1 |
| D5 | What counts as success? | leaning — refusal accuracy | confirm |
| D6 | Notebook the user runs, or executed result? | leaning — notebook, as checked policy | confirm |
| D7 | Publish a recipe convention, or consume one? | leaning — draft it, demo one | scoped to a 1-page proposal |
| D8 | Which vocabulary for the record? | leaning — GeoCroissant + extension | ask Rajat |
| D9 | How does the librarian obtain a record? | leaning — tool for the loop, resource for citation | out of scope this week |
| D10 | RAG over docs, or extraction into a schema? | leaning — both, split by job | confirm |
| **R1** | Can fitness knowledge be extracted at all? | **unknown** | day-one probe, 3 datasets |
| R2 | Scope absorption | open | posture, not a task |

**Only D1 blocks work.** Everything else has a leaning that is safe to proceed on; a
"confirm" needs sixty seconds in a room, not a discussion.

### Before anyone codes

#### D1 — Which region, question and persona?
**Status:** open · **needs an owner** · *a defensible default is proposed below*

Fixtures, gold set and demo script all descend from this, and none can start without
it. Owner should be whoever owns the science framing. Assign this first, not last.

##### A default that can be adopted or rejected in five minutes

**Boulder County, Colorado. Primary persona: the land manager. Refusal case: the snow
observer.** Not an aesthetic choice — it is where the evidence already is:

- The land-manager query is **already run and verified** against the live NASA MCP:
  287 collections, top five at 9 / 9 / 9 / 3 / 36 km. The demo's opening beat exists.
- **Four of the five records are already half-authored.** We have pulled UMM-C for
  HLSL30 (30 m), SPL3SMP (36 km), MOD16A2 (500 m) and GPM_3IMERGHH (resolution block
  absent). Day one starts from evidence, not setup.
- **One region carries both personas.** The plains east of Boulder give the land manager
  a real agricultural question; the Front Range west of it gives the snow observer a real
  slope question — which is the case that must be *refused*, because operational snow
  products are coarse. That is exactly the pairing D4 wants.
- **The terrain does double duty.** Boulder County is half mountainous, which is where
  SMAP retrieval degrades — so the *retrieval validity* axis has a real instance, not a
  hypothetical one.
- The memo's own recipe example is already written against this bounding box:
  `(-105.30, 39.90, -105.10, 40.10)`.

##### What would justify overriding it

**Pick where the expert is, not where the evidence is, if they conflict.** The gold set
needs correct answers authored by someone who knows the products well enough to be
trusted as ground truth. If someone in the room knows a different region or product
family far better, that beats Boulder — re-running the probes costs an hour, and
authoring five records without real expertise costs the whole validation.

##### Criteria for any alternative

Whatever is chosen must satisfy all five, or the week's validation does not work:

1. **Candidates genuinely disagree on an axis** — otherwise no interview question has
   leverage and the disagreement-driven interview cannot be demonstrated.
2. **A plausible-but-wrong option exists** — the trap. (Here: IMERG, which sounds right
   and is 11 km.)
3. **At least one question is genuinely unanswerable at the scale asked** — without this
   there is nothing to score refusal accuracy against.
4. **Records are cheap to author** — someone in the room must know these products.
5. **It reuses evidence already gathered**, or budgets an hour to regather it.

#### D2 — Which providers may the librarian search?
**Status:** leaning — three

CMR-STAC is partitioned per DAAC; there is no global granule search, so the shelf is an
explicit list rather than a wildcard. Wide looks good on a slide and does not finish in
a weekend.


### Before the demo

#### D4 — Which role goes on stage?
**Status:** leaning — open with the refusal, success is the second act

The land manager succeeds. The snow observer gets declined. Only one is hard to fake.

#### D5 — What counts as success?
**Status:** leaning — refusal accuracy

Candidates: reproducing the chart's recommendations, materialising a cube, or refusal
accuracy. Each instruments differently. Refusal accuracy is the only one a demo that
always says yes cannot fake.

#### D6 — Notebook the user runs, or executed result?
**Status:** leaning — enforce *reference, not advice* as a checked policy, not a prompt
instruction

The notebook keeps a human in the loop by construction. The executed result demos
better and is far easier to over-trust.

#### D7 — Do we publish a recipe convention, or just consume one?
**Status:** leaning — draft the convention, demo one collection that conforms

A `role: example` asset with CI-verified snippets is proposable to STAC and to the
DAACs — an artifact that outlives the weekend. A spec plus one instance beats a
prototype.

**Note 2026-08-25.** This is about the *recipe*. The *record's vocabulary* is now a
separate question — see D8, which has a much stronger venue than a from-scratch STAC
convention.

#### D8 — Which vocabulary for the readiness record?
**Status:** leaning — **adopt GeoCroissant; propose the fitness block as an extension**

Opened 2026-08-25 after Rajat pointed at Croissant. See
[notes/2026-08-25-croissant-geocroissant.md](notes/2026-08-25-croissant-geocroissant.md).

**Rajat Shinde (NASA IMPACT / UAH) leads GeoCroissant** — spec 1.0, published
2026-01-29. He authored the standard he pointed at, so adoption costs nothing in
negotiation and extension is a live option rather than a hopeful proposal.

GeoCroissant closes the **Shape** block properly. `geocr:spatialResolution` is a
`QuantitativeValue` — a number with a unit, defined as ground sampling distance per
pixel — which is exactly the normalised field CMR lacks. Also
`geocr:coordinateReferenceSystem`, `geocr:temporalResolution`,
`geocr:bandConfiguration`, `geocr:recordEndpoint`.

It does **not** close the **Fitness** block. No `quality_flag_convention`, no
`uncertainty`, no `min_meaningful_area_km2`, no `known_artifacts[]`. The nearest
neighbours are free text aimed at a different question: `geocr:spatialBias` means a
training set under-samples a region; `rai:dataUseCases` is an ML-lifecycle enum
(Training / Testing / Validation), not science use. The **Recipe & verdict** block has
no counterpart at all.

So: use their names where they exist, extend where they don't, via the spec's own
escape hatch (`sc:additionalProperty` / `sc:PropertyValue`, or a namespace of ours).
**Do not invent parallel names for fields GeoCroissant already defines.**

Ask Rajat: is a fitness extension welcome, and what populates `geocr:spatialResolution`
when the source STAC collection does not carry it?

#### D9 — How does the librarian obtain a record?
**Status:** leaning — **an MCP tool for the loop, a resource for citation; both from one
record**

Opened 2026-08-25. The three MCP primitives differ by *who controls them*:

| Primitive | Who decides | Context cost |
|---|---|---|
| **Tool** | the **model**, mid-reasoning | description sits in context **every turn** |
| **Resource** | the **user or client**, attached deliberately | nothing until read |
| **Prompt** | the user, as a slash command | nothing until invoked |

The ranking loop is model-driven — the librarian pulls records for five candidates
while comparing them — so it needs a **tool** (`get_readiness(concept_id)`). Resources
are application-controlled and will not serve that. A **resource**
(`readiness://collection/{concept_id}`) is right for citation, browsing, and a methods
section.

The token economics favour this: one tool description is a few hundred tokens standing,
and records load only for datasets actually under consideration — against NASA's
4,812-token instructions block, which every request pays for and which carries exactly
one hardcoded dataset caveat.

**The point that matters: NASA's server exposes no readiness primitive of any kind** —
no tool and no resource. The slot is empty in both forms.

#### D10 — RAG over documents, or extraction into a schema?
**Status:** leaning — **both, with a strict division of labour**

Opened 2026-08-25 after [AquiLLM](https://arxiv.org/abs/2508.05648) (Campbell, Boscoe &
Do) — a RAG system for capturing research groups' **tacit knowledge**. See
[notes/2026-08-25-aquillm.md](notes/2026-08-25-aquillm.md). SlideRule made the same
choice the other way, serving docs behind a retrieval endpoint.

We had made this choice implicitly. Make it explicit:

| | Retrieval index | Readiness record |
|---|---|---|
| Ingest cost | very low | high — each field authored |
| Preserves nuance | verbatim | lossy by design |
| Arbitrary questions | yes | only what the schema anticipated |
| **Computes a comparison** | **no** | **yes** |

**The deciding property:** retrieval can surface the paragraph saying *"above 80°N,
multiple overpasses can be gridded into a single MGRS tile."* It cannot compare 80°N to
a bounding box. The fitness axes are comparisons, not lookups — so the ranking loop needs
the record.

But the record only holds what we thought to define. So: **schema for the ten axes;
retrieval for the long tail and as the authoring surface for the `authored/` tier.**
Keep the corpora separate — mixing private group material with untrusted DAAC prose in
one index puts two legs of the lethal trifecta in the same context.

### Open risk

#### R1 — Where does fitness knowledge actually come from?
**Status:** unknown · **test on day one**

Can a model pull resolution, quality-flag conventions and minimum meaningful area out
of DAAC documentation reliably enough to act on — or must the demo set be curated by
hand? This is the load-bearing assumption of the whole design, and the cheapest thing
to test. Try three datasets.

**Narrowed 2026-08-25.** The NASA MCP's `get_variables` (UMM-V) already returns
`scale`, `offset`, `fill_values`, `valid_ranges` and `units`; `get_services` returns
access endpoints. So parts of identity and access come free and structured from CMR.
`get_collections` also returns `spatial_resolution` and `temporal_resolution` — but as
free text in mixed units (`"36.0x36.0 Kilometers"` vs `"30x30 Meters"`), and **`null`
for GPM_3IMERGHH**, the one dataset our land-manager persona must be prevented from
accepting.

**Corrected 2026-08-25 (later).** The free text is the *MCP's* rendering. UMM-C carries
resolution structured — `{"XDimension": 30, "YDimension": 30, "Unit": "Meters"}` — and
also defines `Quality` and `Purpose`. So R1 splits into three jobs: **author** what is
absent (IMERG has no resolution block at all), **normalise** what is prose (HLSL30's
`Quality.Summary` states a ≥ 80°N caveat no agent can compare to a bbox), and
**preserve** structure the agent layer currently flattens. See
[notes/2026-08-25-nasa-ontology-slot.md](notes/2026-08-25-nasa-ontology-slot.md).

**None of the fitness fields are available at all** — `quality_flag_convention`,
`uncertainty`, `min_meaningful_area_km2`, `known_artifacts[]` and `cautions[]` remain
ATBD/DAAC-guide territory. Test extraction against the fitness block only; don't spend
day one re-deriving UMM-V.

**And note what D8 does not change.** GeoCroissant defines `geocr:spatialResolution` as
a proper quantity, which fixes the *schema* problem. It does not populate it. Croissant
is a vocabulary, not a corpus — IMERG's `null` does not become 11 km because a field
for it exists somewhere. R1 is a question about **producing values**, and adopting a
better vocabulary leaves it exactly where it was.

#### R2 — Scope absorption
**Status:** open · opened 2026-08-25

Three agent-layer efforts are already in flight: NASA's `earthdata-mcp` (pushed four
days ago, its instructions visibly growing to absorb this kind of guidance), **ChatGSFC**
(NASA-internal, 9,000+ users, Jason Gilman), and **Element 84**'s general-purpose
geospatial agent. A fourth agent is not a contribution.

None of them builds the characterisation layer. So bet on artifacts a system prompt
cannot hold — the record and the vocabulary extension (D8), not better agent behaviour.
Anything we build as *a smarter agent* is on a collision course; anything we build as
*the operand nobody supplies* is not.

---

## Decided

#### D3 — Existing Earthdata MCP server, or reimplement access?
**Status:** effectively settled 2026-08-25 — **be a client of `nasa/earthdata-mcp`**

Aimee's resources reference one. If it works, the librarian is a client rather than an
integration. It removes plumbing, not judgement.

Resolved by probing it live during Joe Hamman's MCP talk — see
[notes/2026-08-25-joe-hamman-mcp.md](notes/2026-08-25-joe-hamman-mcp.md).
`https://cmr.earthdata.nasa.gov/mcp/v1` is hosted by NASA, on streamable HTTP, pushed
four days ago, exposing seven CMR tools — and **discovery needs no authentication**.
Earthdata Login is required only for granule download. Two rival servers
(datalayer, podaac) are four to five months stale.

Confirm with Aimee that this is the server her resources point at, then treat it as
settled. One caveat before we depend on it: **the repo declares no license.**

---

## Template for new decisions

## [YYYY-MM-DD] - Decision Title

### Context
What is the issue that we're seeing that is motivating this decision?

### Decision
What is the change that we're proposing?

### Consequences
What becomes easier or more difficult to do because of this change?
