# Project: NASA Librarian Agent

## Objective
Health researchers and other users access and use NASA Earth data more effectively
using generative AI.

The system is a reference desk for NASA Earth data: it interviews the user, finds
what fits, hands back code that runs — and says plainly when nothing in the holdings
answers the question at the scale asked.

## The gap being addressed

**The catalog describes what a dataset *is*. Fitness is a relation between a dataset
and a question. Nothing in the system holds the dataset-side facts needed to compute
that relation — so no agent can, however good it is.**

### Current state

A land manager asks: *is my county drying out?* Searching NASA's catalog for soil
moisture over Boulder County returns **287 collections**. The top five, in rank order:

| | grid |
|---|---|
| SPL4SMGP | 9 km |
| SPL3SMP_E | 9 km |
| SPL4SMAU | 9 km |
| SPL2SMAP_S | 3 km |
| SPL3SMP | **36 km** |

All five are returned as matches. All five are genuinely *available* — granules exist
for that box and time. The last has a single pixel wider than the county. His parcels
run to ~40 ha, about 630 m across; the top result's pixel is 14× wider than that and
200× the area.

Nothing in the response says so. He takes the top hit, gets a time series, and it is a
real number that means nothing about his fields. **He never learns he asked a question
this data cannot answer.**

*(Verified 2026-08-25 against the live NASA Earthdata MCP — unauthenticated, one call.
See [notes/2026-08-25-joe-hamman-mcp.md](notes/2026-08-25-joe-hamman-mcp.md).)*

### What is missing, precisely

Resolution comes back as free text in mixed units — `"36.0x36.0 Kilometers"`,
`"30x30 Meters"`. Parseable, not comparable.

For **IMERG** — the classic wrong answer, the one that sounds right because it is named
"precipitation" — it is **`null`**. Not wrong; absent. Its 0.1° grid appears only in the
free-text title, and converting that to ~11 km requires knowing the latitude.

And resolution is the *easy* field. Quality-flag conventions, uncertainty, minimum
meaningful area, discouraged uses — **none of these have a field at all**. NASA needed
exactly one such fact (HLS does not populate CMR's `cloud_cover`; use the Fmask band)
and hard-coded it into their MCP server's system prompt, for that one dataset.

So the knowledge exists. It lives in ATBDs, DAAC guides, hand-written special cases in
system prompts, and in people who have used a product for a decade. **It has no
machine-readable home.**

### What this week unlocks

The readiness record is the **operand**. The librarian is the **operator**. Without the
operand there is nothing to compute on — which is why the week weights toward the
record.

Characterise even five datasets and the same query returns:

> HLS at 30 m resolves your parcels at ~20 px across. SMAP is county context only —
> 9 km pixels cannot separate your fields. IMERG cannot answer this at all: one pixel
> exceeds your county.

**The unlock is that "no" becomes computable.** Today no agent can say it, because the
facts required are not anywhere.

### What fitness means, beyond relevance

Relevance is a property of the dataset alone — do its indexed words match the words in
the question? It yields an *ordering*, so there is always a top result. That is why 287
hits arrive with a number one and nothing that says stop: **a ranking cannot return
nothing.**

Fitness is a predicate over **(dataset, question, place, time)** and can be false for
every candidate. It is *non-monotone in relevance* — IMERG is more relevant to the word
"precipitation" than HLS is, and less fit for the question — so sorting harder never
finds it out.

And it is not one comparison but about ten, failing independently:

| Axis | Record supplies | Interview supplies | Failure it catches |
|---|---|---|---|
| **Can it see the thing?** | | | |
| Spatial resolvability | `native_resolution_m`, `min_meaningful_area_km2` | unit of analysis | 9 km pixel vs 630 m parcel |
| Temporal cadence | `temporal_cadence` | rate of the process | 8-day composite vs flash drought |
| Record length | `time_start`, `time_end` | window the trend needs | 5-year mission vs 30-year question |
| Latency | `latency` | operational or retrospective | 3-month lag on a "right now" question |
| **Does it measure the thing?** | | | |
| Semantic match | `measured_variable`, `validated_uses[]` | what the word means here | "precipitation" ≠ "dryness"; surface soil moisture ≠ root zone |
| Retrieval validity | `known_artifacts[]`, `cautions[]` | where | SMAP degrades over mountains and dense vegetation — half of Boulder County |
| Detectability | `uncertainty` | expected effect size | retrieval error larger than the trend |
| **Can it be used correctly?** | | | |
| Handling obligation | `quality_flag_convention` | — always applies | unmasked HLS quietly ruins a time series |
| Joint usability | `crs`, grid, overpass time | which datasets combine | two fit datasets that will not co-register |
| Practical reach | `format`, volume, `protocol`, `auth` | compute and skill on hand | 2 TB for one county-decade |

Three consequences:

1. **Fitness is per-set, not only per-dataset.** Joint usability is a property of the
   combination, and the deliverable is a co-registered cube — so the desk must judge
   pairs, not just rank singles.
2. **Fitness is place-dependent.** The same product is fit in Kansas and unfit in the
   Rockies. A global verdict is not enough; `cautions[]` must be able to carry regional
   conditions.
3. **Half the operands come from the person asking.** Which is why the desk interviews
   before it answers, and why the record must exist before the desk can. Relevance needs
   one operand and the catalog has it. Fitness needs two, and today *neither* side is
   written down — the dataset half lives in ATBDs, the question half is never elicited.

### Two kinds of refusal — do not blur them

They look alike and are not. NASA's MCP already does the first, and does it well.

| | Refuses when | Status |
|---|---|---|
| **Availability** | No granules for this box and time | **Solved.** NASA's server is strict about it: don't broaden filters, zero is the correct answer |
| **Fitness** | Granules exist, but the data cannot answer at this scale | **Nobody does this.** NASA explicitly disclaims it (see below) |

### Corroboration

Three independent confirmations, all from NASA or its authors:

1. **The chart doesn't ask the question.** [*Which data tool is right for you?*](earthdata-tools-chart_v2_072025_1.pdf)
   (v2, July 2025) scores ten tools across nineteen rows. Every row asks what the
   **tool** can do; none asks whether the **data** is fit for the asking. It will tell
   a land manager that AppEEARS can subset for a point; it cannot tell them the product
   they are subsetting has an eleven-kilometre pixel.
2. **NASA's MCP server declines to answer it**, in its own instructions: asked for the
   "best" data or "highest quality" metadata, the agent "must immediately inform them
   that the tools do not support sorting, filtering, or evaluating by qualitative
   metrics."
3. **When an agent answers anyway, the expert disagrees.** Joe Hamman (CTO, Earthmover)
   demoed exactly this on 2026-08-25 — asked for the *best* sea-surface-temperature
   repo across 195 of his own, got a pick with a justification, and did not accept it.

Ten tools fit on one page and a person maintains them. Collections number in the
thousands, change continuously, and their fitness depends on the question asked — so
the same table one level down has to be generated rather than authored. That is the
argument for an agent here.

## Success Criteria
- [ ] Make NASA data sets more discoverable by LLMs and by a NASA librarian agent
- [ ] Build a NASA librarian agent that helps researchers find and access data for
      their research goals
- [ ] Provide initializing code to get researchers started using NASA data

### How these are measured
Refusal accuracy is the primary metric — the only proposed measure a demo that always
says yes cannot fake. See [decisions.md](decisions.md) D5.

## Architecture
Three proposals, three functions of one library, meeting in a single shared record.
Remove the record and there are three demos that happen to mention the same datasets.

| Function | What it does | Owner |
|---|---|---|
| **Technical services** — readiness assessment | Two skills (cloud-readiness, AI-readiness) turning a dataset + catalog entry into a structured verdict | Aimee |
| **Reference desk** — librarian agent | Interviews, searches, ranks against the stated need, reports what won't work and why | James |
| **Circulation** — access recipe | Turns a chosen dataset into a cell that runs, with auth, subsetting and quality-flag handling written in | Keenan |

Inputs are NASA holdings (CMR-STAC catalogs, DAAC guides, ATBDs). Output is a
co-registered cube plus a compatibility report — or a documented refusal.

Catalog access is **not ours to build**: the librarian is a client of NASA's hosted
`earthdata-mcp` (see decisions.md D3). We build the layer that server says it cannot
do.

### The Dataset Readiness Record
The shared contract. "Ready" means three things, produced at different times:

| Layer | Question | Produced by |
|---|---|---|
| Cloud-ready | Can a machine fetch this efficiently? Format, chunking, compression, access-pattern/chunk-layout match | Aimee · cached |
| AI-ready | Is this characterised well enough to use responsibly? Resolution, uncertainty, provenance, quality flags, discouraged uses | Aimee · cached |
| Fit for the question | Does it answer *this* question at *this* scale? | librarian · per query |

The first two cache; the third cannot exist until someone asks. So the two skills write
**one record, never two** — reconciling documents at query time is work the librarian
should never do.

Field groups: identity (`concept_id`, `short_name`, `version`, `doi`, `provider`),
access (`protocol`, `auth`, `endpoints[]`), shape (`format`, `structure`, `crs`,
`native_resolution_m`, `temporal_cadence`, `latency`), fitness
(`quality_flag_convention`, `uncertainty`, `known_artifacts[]`,
`min_meaningful_area_km2`, `validated_uses[]`, `cautions[]`), recipe & verdict
(`access_recipe`, `recipe_last_verified`, `cloud_ready`, `ai_ready`, `blockers[]`).

#### Vocabulary: GeoCroissant, extended

Use **GeoCroissant** names where they exist rather than inventing parallel ones — spec
1.0, published 2026-01-29, **led by Rajat**. See decisions.md D8.

| Our field | GeoCroissant | Type |
|---|---|---|
| `crs` | `geocr:coordinateReferenceSystem` | structured, EPSG |
| `native_resolution_m` | `geocr:spatialResolution` | **`QuantitativeValue` — number + unit** |
| `temporal_cadence` | `geocr:temporalResolution` | `QuantitativeValue` |
| endpoints | `geocr:recordEndpoint` | OGC API – Records URL |

That closes the **shape** block properly, and `geocr:spatialResolution` is precisely the
normalised quantity CMR does not carry.

The **fitness** block has no GeoCroissant counterpart — `quality_flag_convention`,
`uncertainty`, `min_meaningful_area_km2` and `known_artifacts[]` are all absent, and
the near neighbours aim elsewhere (`geocr:spatialBias` is about training-set
representativeness; `rai:dataUseCases` is an ML-lifecycle enum). Neither does the
**recipe & verdict** block. **That absence is the project's contribution** — proposed
as a GeoCroissant extension, which is a far better venue than a from-scratch
convention.

Earlier lineage: the ESIP AI-ready data checklist. **The trap in adopting either
literally:** every ESIP item is a *presence* question. The librarian doesn't need
presence, it needs the value. "Resolution is documented" is useless; `30 m, from the
ATBD, high confidence` is the ballgame. Emit `{value, source, confidence}` wherever a
value exists.

**And the trap one level up:** a vocabulary is not a corpus. `geocr:spatialResolution`
being defined does not populate it — IMERG's `null` in CMR does not become 11 km
because a field for it exists somewhere. Producing values is still the work.

#### Size, and how it is served

The record is a **compact verdict, not a metadata dump**. The librarian holds five to
ten at once while ranking, on top of ~10,900 tokens of MCP tool definitions and
paginated catalog responses. If the record grows to the size of a UMM-C record we have
rebuilt the problem.

Served two ways from one record (decisions.md D9): an MCP **tool**
(`get_readiness(concept_id)`) for the ranking loop, because that is model-driven; and
an MCP **resource** (`readiness://collection/{concept_id}`) for citation and browsing,
because resources are user- and client-controlled. NASA's server exposes **neither** —
no readiness primitive of any kind.

### The reference desk — how the gap gets filled

#### Three artifacts, not two

GeoCroissant's *subject* is a dataset. The interview's output is about a **question**.
Putting question-side facts into a GeoCroissant document asserts properties of the
dataset that belong to the asker — a category error that would poison the cache, since
the record is shared across users and the need is not.

```
   NEED STATEMENT                      READINESS RECORD
   per query · ours                    per dataset · Aimee's · GeoCroissant
   ─────────────────                   ────────────────────
   unit of analysis                    geocr:spatialResolution
   time window + cadence               quality_flag_convention
   what "drying" means here            cautions[] · uncertainty
   effect size · operational?          min_meaningful_area_km2
            │                                    │
            └────────────────┬───────────────────┘
                             ▼
                   COMPATIBILITY REPORT
                   per query · chosen, rejected, and why
```

GeoCroissant is right for the readiness record, **wrong for the need statement**, and
the compatibility report is a third small thing that cites both. That report is also the
deliverable for the researcher persona — it is what goes in a methods section.

#### The need statement — six slots

| Slot | What it pins down |
|---|---|
| `unit_of_analysis` | parcel, field, county, watershed — the thing that must resolve |
| `time_window` | the span a trend or comparison needs |
| `cadence_required` | how often, set by the rate of the process |
| `variable_meaning` | what the user's word denotes here — "drying" as soil, vegetation or rainfall |
| `effect_size` | the change that would matter, which sets the detectability bar |
| `latency_tolerance` | operational (needs recency) or retrospective |

#### The interview is slot-filling, driven by disagreement

Not "the agent interviews the user." Precisely:

**Every input channel fills slots. The desk asks only about slots that are still empty
*and* would change the answer.**

That second clause is the mechanism. Given N candidates with known dataset-side
operands and a partly-filled need statement, for each empty slot ask whether the
candidate set changes across plausible values — then ask about the **highest-leverage
slot first**, and stop when the answer is stable.

For *is my county drying out*: candidates disagree wildly on spatial resolvability
(30 m to 36 km), so **ask about unit of analysis first** — one question eliminates most
of the 287. They barely disagree on latency, so don't ask. Two questions, not ten.

**The refusal falls out for free.** If every candidate fails an axis regardless of the
answer, don't ask — refuse and explain. That is the snow-observer case: the desk does
not interview its way to a no, it detects that no answer helps.

Implementable in a week over five datasets, and no incumbent does it — a ranking system
has nothing to be uncertain *about*.

#### Input channels — same slots, different fill rates

| Channel | Fills | Persona |
|---|---|---|
| Question text + bbox | unit of analysis and time window, partially | everyone |
| **A prototype notebook** | the densest channel — clipping to a shapefile reveals the unit of analysis, a resample call reveals cadence, the statistic reveals effect size. *Evidence of intent* rather than stated intent | researcher |
| **Interview questions** | whatever the others left empty | the only channel for the land manager, who has no notebook |

A notebook is a fast path, never a requirement.

#### How the record attaches to the dataset

Three stages, and GeoCroissant already supplies the hook:

1. **This week** — sidecar files in this repo, keyed by `concept_id`. Unblocks everything today.
2. **For the demo** — served from our MCP as `get_readiness(concept_id)`, consumable by any agent (D9).
3. **The proposal** — the record travels *with* the collection. `geocr:recordEndpoint` is
   defined as a URL pointing to a service where dataset metadata records can be queried
   programmatically. That is the attachment point, already in Rajat's spec (D7, D8).

## Why the catalog channel, not the documentation channel
An agent already knows a great deal about NASA data, absorbed in pretraining. That
prior has two properties nobody chose:

| | The open web | The catalog |
|---|---|---|
| Source | Tutorials, repos, papers, blogs, library source | STAC collection metadata, assets, links, read at query time |
| Freshness | Frozen at training cutoff | Current by construction, and checkable |
| Bias | Popularity — the most written-about product wins, fit or not | None inherent; conformance is binary |
| Who can influence it | Whoever can publish and promote for years | Any producer who publishes a conformant asset — a new mission competes on day one |

Making data discoverable through the first channel means writing more documentation and
hoping it is read, which rewards the already-popular and cannot help a mission that
launched last year. The catalog channel has no such property. It is the fix available
to a small DAAC, and the one that stays true after the next model is trained.

**Amendment, 2026-08-25.** With a live MCP server in the loop the agent is no longer
working from pretraining — it queries the catalog. So the failure we can now
demonstrate is *not* popularity bias; by this table's own account the catalog channel
has none. It is **fitness blindness**: the catalog is unbiased and uncharacterised.
That is a sharper claim and a better-defended one, and it is what the gold set should
measure. Popularity bias in the unaided agent remains worth measuring, but as a
secondary result — a user with the MCP is not unaided.

This is why the access recipe belongs **in the collection** — a link or asset with a
role like `example`, four lines with pinned versions, beside a plain-markdown usage doc
at a stable URL. And every collection's snippet runs in **scheduled CI**: a recipe that
passed an hour ago is a fact; a recipe in a PDF is a hope.

## Personas
| Persona | Ask | The desk must prevent |
|---|---|---|
| Land manager | "Is my county drying out?" | Accepting IMERG because it is named "precipitation" — one pixel is larger than the question |
| Snow observer | "How much snow is left on this aspect?" | Becoming an avalanche decision aid. The desk answers about data, never about whether a slope is safe |
| Researcher | "Burn severity against vegetation recovery in this watershed" | An unreproducible result — this user needs the provenance trail more than the recommendation |

The snow observer most likely gets *nothing suitable* (operational snow products are
coarse), plus the closest available and what it cannot show.

## Timeline
- Started: 2026-08-24
- Target completion: 2026-08-31
- Status: active

## Context
Collaborative project started at the Responsible Generative AI workshop hosted by the
UW eScience Institute, August 2026. Drafted with Keenan, Aimee, Sarah, Julia and Rajat.

Architecture memo (source of this document):
https://claude.ai/code/artifact/37a5c8c4-ee37-4c90-a97b-f9cdaa7f9dd5

## Next Actions
Four tracks that can run in parallel:

- [ ] **Baseline against the MCP-equipped agent, not an unaided one.** Before any skill
      exists, run an agent *with the NASA Earthdata MCP connected* at the gold questions
      and save what it picks. This is the incumbent, and the first question any reviewer
      asks is whether we compared against the official server. Seed the set with cases
      where the best-fit product is obscure and a plausible one is wrong-scale — the
      failure to demonstrate is fitness blindness, not popularity bias. Keep the
      unaided-agent run as a secondary result.
- [ ] **Hand-write three readiness records** before lunch on day one. The librarian
      develops against those; Aimee's generator replaces them when it lands.
- [ ] **Build a fifteen-question gold set**, including cases where the right answer is
      that NASA holdings cannot answer at the scale asked. Scoring the refusals is what
      separates this from a demo that only ever succeeds. **Score the two refusal types
      separately** — availability-refusal is already solved by NASA's server, so mixing
      them in means claiming credit for finished work. Include at least one unrefined
      first prompt where the correct behaviour is a clarifying question, not a ranking.
- [ ] **Keep one known-good path warm** — one region, one product pair already
      co-registered. Don't show the fallback; don't be without it.

## Dependencies
- The science framing (region, question, persona) gates fixtures, gold set and demo
  script — see decisions.md D1. Assign first, not last.
- **Rajat** on GeoCroissant: whether a fitness extension is welcome, and what populates
  `geocr:spatialResolution` when the source STAC collection does not carry it (D8).
- `HarshShinde0/geocr_mcp` — a GeoCroissant MCP server pushed 2026-08-25 that generates
  GeoCroissant JSON-LD from STAC searches. A *producer* tool, and plausibly the machine
  that mass-produces Aimee's records. No license declared.
- Whether the existing Earthdata MCP server works determines if the librarian is a
  client or an integration (D3).
- The librarian develops against Aimee's readiness records; hand-written fixtures
  unblock this until the generator lands.
- Earthdata Login credentials for any recipe that actually runs.

## Open risk
Can a model pull resolution, quality-flag conventions and minimum meaningful area out
of DAAC documentation reliably enough to act on — or must the demo set be curated by
hand? This is the load-bearing assumption of the whole design and the cheapest thing to
test. Try three datasets on day one.
