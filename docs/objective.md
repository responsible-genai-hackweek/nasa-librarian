# Project: NASA Librarian Agent

## Objective
Health researchers and other users access and use NASA Earth data more effectively
using generative AI.

The system is a reference desk for NASA Earth data: it interviews the user, finds
what fits, hands back code that runs — and says plainly when nothing in the holdings
answers the question at the scale asked.

## The gap being addressed
NASA already publishes a decision aid — [*Which data tool is right for you?*](earthdata-tools-chart_v2_072025_1.pdf)
(v2, July 2025), ten tools scored across nineteen rows. Every row asks what the **tool** can do.
No row asks whether the **data** is fit for the asking. The chart will tell a land
manager that AppEEARS can subset for a point; it cannot tell them the product they are
subsetting has an eleven-kilometre pixel.

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

Vocabulary follows the ESIP AI-ready data checklist, which gives the schema a citable
lineage. **The trap in adopting it literally:** every ESIP item is a *presence*
question. The librarian doesn't need presence, it needs the value. "Resolution is
documented" is useless; `30 m, from the ATBD, high confidence` is the ballgame. Emit
`{value, source, confidence}` wherever a value exists.

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

- [ ] **Measure the popularity bias, don't assert it.** Before any skill exists, run a
      plain agent at the gold questions and save what it picks. Seed the set with cases
      where the best-fit product is obscure and a heavily-documented one is plausible
      but wrong-scale. If the unaided agent reaches for the famous product, that is a
      demonstrated bias rather than a claim about training data — and it is the result
      the whole project exists to answer.
- [ ] **Hand-write three readiness records** before lunch on day one. The librarian
      develops against those; Aimee's generator replaces them when it lands.
- [ ] **Build a fifteen-question gold set**, including cases where the right answer is
      that NASA holdings cannot answer at the scale asked. Scoring the refusals is what
      separates this from a demo that only ever succeeds.
- [ ] **Keep one known-good path warm** — one region, one product pair already
      co-registered. Don't show the fallback; don't be without it.

## Dependencies
- The science framing (region, question, persona) gates fixtures, gold set and demo
  script — see decisions.md D1. Assign first, not last.
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
