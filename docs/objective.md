# Project: NASA Librarian Agent

## Objective
Earth science researchers, land and resource managers, and other users outside a
mission's expert community access and use NASA Earth data more effectively using
generative AI.

The system is a reference desk for NASA Earth data: it interviews the user, finds
what fits, hands back code that runs — and says plainly when nothing in the holdings
answers the question at the scale asked.

**Stated as a role:** we are automating the most-repeated part of what a research
software engineer, a DAAC user-services team, or a science data librarian does — the
question-driven triage that decides *which* data can answer *this* question, and how it
must be handled. See [Who does this today](#who-does-this-today).

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

The facts needed to catch this are missing in three different ways — absent, unstructured,
or flattened in transit. [design.md](design.md#the-failure-in-full) has the evidence.

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

### Who does this today

<a id="who-does-this-today"></a>

A person. Specifically: a **research software engineer** embedded in a lab, a **DAAC
user-services team**, or the colleague who has used a product for a decade and can tell
you in one sentence why it won't work for you.

**The tacit knowledge envelope.** Every mission has a core team holding this knowledge.
Researchers *adjacent* to that team — a lab over, a collaborator, someone who attended
the workshop — become flexible, capable users of the mission's data. Researchers
*outside* the envelope largely cannot extract value from it, no matter how open the
archive is. **Openness is not access.** The project's target is the population outside
the envelope, and the measure of success is how far the envelope extends without adding
people to the mission team.

J.P. Swinski (SlideRule, 2026-08-25) described his own job in exactly these terms:
taking the scientific ideas of Tyler Sutterley and David Shean and delivering them
either as **best-practice default options** or through **a system that asks questions
leading to good choices** — and SlideRule as a technology push to *reduce the scientific
insight required from ICESat-2 experts and enable wider use of the data*. Those are our
two mechanisms, named independently: defaults baked into the recipe, and the
disagreement-driven interview.

That is the clearest statement of what this project is for. Not a better search engine —
an attempt to make **one specific, scarce, expert judgement reproducible at scale.**

Those people are:

- **Scarce and expensive.** Every DAAC has a handful; most labs have none.
- **Not scalable.** The triage is delivered one email, one office hour, one Slack thread
  at a time.
- **Undocumented.** Their knowledge is the thing we keep saying "lives in people who have
  used a product for a decade." The literature calls this **tacit knowledge** — informal,
  experience-based expertise passed down orally through meetings, mentoring and daily
  collaboration ([Campbell, Boscoe & Do 2025](https://arxiv.org/abs/2508.05648)). When
  they retire it leaves with them.
- **Answering the same question repeatedly** for different users with similar needs.

That last point is the opening. The judgement is *per question*, but the **facts it rests
on are per dataset** — and those get re-derived from scratch every time, by a person,
from an ATBD.

> **The readiness record is where that expert's knowledge gets written down once,
> instead of being re-delivered by email fifty times.**

We are not replacing them. The record is the artifact where the repeated part of their
work becomes reusable, and the desk is what applies it when they are not in the room.

#### What that role actually delivers — and what we are copying

| What the expert provides | Our artifact |
|---|---|
| "Use HLS, not SMAP, for parcel-scale work" | compatibility report — the chosen dataset with the axis that decided it |
| "SMAP won't resolve your fields, but it's fine as county context" | the *rejected* entries, with the axis that killed each |
| "Nothing in the holdings answers that at the scale you're asking" | the refusal, as a first-class output |
| "Mask with Fmask or you'll ruin the time series" | caveats carried into the access recipe |
| "This product has been validated for X, not Y" | `validated_uses[]` / `cautions[]` |

Note that **"usage patterns" is two different things**, both in scope and produced by
different people: `validated_uses[]` is *what this data has been used for successfully*
(Aimee, cached, per dataset); the access recipe is *how to touch it correctly* (Keenan,
per selection).

#### Two things that framing must not obscure

**1. The record holds operands, not answers.** The fields hold *dataset properties* —
facts true regardless of who asks. Nothing is pre-computed. *Fit for the question* cannot
exist until someone asks; if answers were stored you would need one per
(dataset × question), which is unbounded. The design depends on caching the operands and
computing the relation fresh.

**2. Half the input comes from the user, not from metadata.** No amount of metadata
enrichment produces guidance on its own, because *"is 9 km too coarse?"* has no answer
until someone says what they are measuring.

```
DATASET SIDE (cached · Aimee)           QUESTION SIDE (per query · the interview)
UMM-C Quality / Purpose / Resolution    unit of analysis · time window
+ ISO useLimitation                     what "drying" means here · effect size
+ the 3 fields nobody defines           operational or retrospective
          │                                          │
          └─────────────► LIBRARIAN ◄────────────────┘
                             │
                             ▼
                COMPATIBILITY REPORT  →  RECIPE
                (the expert's answer, computed per question)
```

This is the half no MCP server, ISO field or vocabulary can supply — **the provider
cannot see the question.** It is also the half the expert gets for free by talking to
you, which is exactly why the desk has to interview.

**The full anatomy** — the ten fitness axes, the record specification, the reference-desk
design and the corroborating evidence — lives in [design.md](design.md). It exceeds what
this week builds, deliberately.

## This week's scope

**One persona, one region, five hand-authored records, and a desk that asks one good
question and refuses when it should — measured against the same agent without it.**

Filling schema fields does not close the gap. The gap is the **comparison and the
conversation**; the records are the operand, the desk is the deliverable. A week that
produces beautiful records and no conversation has closed nothing.

### In

| # | Deliverable | Cost | Owner |
|---|---|---|---|
| 1 | **Fix persona + region + questions** (D1) | 1 hour | *unassigned — blocks everything* |
| 2 | **Five hand-authored readiness records** | half a day | Aimee |
| 3 | **The desk** — interview → compare → compatibility report with chosen *and* rejected, each carrying the axis that decided it | the bulk of the week | Jim |
| 4 | **One recipe that runs**, positive case only, verified by running it once | — | Keenan |
| 5 | **Gold set, ~10–12 questions**, a third unanswerable at the scale asked, answers authored by someone who knows these products. Include at least one unrefined first prompt where the correct behaviour is a *clarifying question*, not a ranking | half a day | domain expert |
| 6 | **Keep one known-good path warm** — one region, one product pair that already works end to end. Don't show the fallback; don't be without it | ongoing | whoever demos |

**The key decoupling is #2.** Records are *authored*, not generated — which makes R1 a
day-one probe rather than a dependency, so the week cannot be sunk by extraction turning
out to be hard.

### Out — stated explicitly so it does not creep back

| Cut | Instead |
|---|---|
| Generating records at scale | Five by hand. R1 probes three datasets on day one; success is a bonus, not the plan. |
| The GeoCroissant extension as code | A **one-page written proposal**. The artifact that outlives the week, at an hour's cost. |
| Serving records over MCP (D9) | Sidecar files keyed by `concept_id`. |
| CI-verified recipes | One recipe, run once, by hand. |
| The co-registered cube | One dataset's recipe running beats two co-registered and half-working. |
| AquiLLM, SlideRule's eval harness, trifecta hardening | Noted in [notes/](notes/). None built. |

### The tension, resolved

The record is the **strategic** contribution — schema and proposal, which is writing.
The desk is the **week's** contribution — which is code. Both matter, on different
clocks. Do not spend build time producing records at scale.

## Success Criteria
- [ ] Make NASA data sets more discoverable by LLMs and by a NASA librarian agent
- [ ] Build a NASA librarian agent that helps researchers find and access data for
      their research goals
- [ ] Provide initializing code to get researchers started using NASA data

### How these are measured
Refusal accuracy is the primary metric — the only proposed measure a demo that always
says yes cannot fake. See [decisions.md](decisions.md) D5.

### The validation — the number that makes this delivered rather than demoed

Run the **MCP-equipped agent** — not an unaided one — on the gold set and record its
picks. Run the desk on the same set. Report:

> How many silently-wrong picks the baseline made, and how many the desk caught.

Target shape: *"the baseline chose a dataset that cannot answer the question in 6 of 12
cases; the desk caught all 6, and correctly refused 4 of 4 unanswerable ones."*

Score the two refusal types separately — availability-refusal is already solved by
NASA's server, so counting it claims credit for finished work.

There is an economy worth noticing: **the expert who authors the gold-set answers is the
same tacit-knowledge source who authors the records.** One person's judgement becomes
both the input and the ground truth — the project in miniature.

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
