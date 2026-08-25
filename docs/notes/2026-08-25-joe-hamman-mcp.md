# Joe Hamman (Earthmover) — MCP

**Talk notes · 2026-08-25 · Responsible Gen-AI for NASA Earthdata, Seattle**

Speaker: Joe Hamman, CTO at Earthmover. Worked on Xarray, Pangeo, Zarr, Xpublish.

> Live notes — taken during the talk, not reviewed. Attributions are as-heard.

## Part one — what MCP is

**The problem MCP solves:** everyone writes the same adaptor between their tools and
an LLM harness, over and over. MCP is one dependable interface for every agent.

**What MCP is**
- An open protocol connecting agents to tools and to data
- SDKs in nearly every language
- A wire format — sends JSON-RPC messages
- A **client-side capability**

**What MCP is not**
- Not a model feature. No model learned MCP in training.
- Not an API gateway
- Not an authentication layer
- It does **not** make the agent use your tool well

There is an earthaccess MCP.

## Architecture

Host → client → server.

Two transports for client → server:
- **stdio**
- **streamable HTTP** — often the more powerful of the two

## Primitives

Tools · resources · prompts

---

# Research: the earthaccess / Earthdata MCP

Prompted by Joe's "there is an earthaccess MCP". Done live during the talk,
2026-08-25. Bears directly on **D3**.

## The landscape — three candidates

| Repo | What it is | Last push | Stars | License |
|---|---|---|---|---|
| **[nasa/earthdata-mcp](https://github.com/nasa/earthdata-mcp)** | Official NASA. Hosted remotely. 7 CMR tools. | **2026-08-21** | 23 | **none declared** |
| [datalayer/earthdata-mcp-server](https://github.com/datalayer/earthdata-mcp-server) | Third-party, built on `earthaccess`, composes with jupyter-mcp-server | 2026-03-19 | 26 | BSD-3-Clause |
| [podaac/cmr-mcp](https://github.com/podaac/cmr-mcp) | PO.DAAC "example", explicitly `earthaccess`-based, stdio | 2026-04-12 | 4 | none declared |

**nasa/earthdata-mcp is the one to use.** Pushed four days ago; the others are
four to five months stale.

## It is live, hosted, and needs no auth to search

Endpoint: `https://cmr.earthdata.nasa.gov/mcp/v1` · **Streamable HTTP** (the
transport Joe called the more powerful one) · server reports `earthdata-mcp v dev`.

Verified by direct JSON-RPC probe — `initialize`, `tools/list` and a real
`get_collections` call all succeeded **anonymously**. Earthdata Login is needed only
for granule *download*, not for discovery. So the librarian can search the catalog
with no credential plumbing at all.

**Seven tools:** `get_keywords` (GCMD vocabulary translation) · `get_collections` ·
`get_granules` · `get_services` (UMM-S: OPeNDAP, Harmony, WMS) · `get_tools` (UMM-T:
Giovanni, Worldview, Panoply) · `get_citations` (papers ↔ datasets, by DOI) ·
`get_variables` (UMM-V).

**Zero prompts, zero resources.** Of Joe's three primitives it ships only tools; all
guidance is packed into the `instructions` string returned by `initialize`.

## Three findings that change our design

### 1. NASA disclaims the fitness question, in their own words

From the server's `instructions`, section **HONESTY AND SYSTEM LIMITATIONS**:

> If a user asks you to perform a qualitative assessment across the catalog — such as
> finding the **"best" data**, the most "complete" records, or the "highest quality"
> metadata — you must: Immediately inform them that the tools do not support sorting,
> filtering, or evaluating by qualitative metrics.

This is the missing row, conceded by NASA in the MCP server itself. Our claim now has
two independent NASA-sourced corroborations: the tools chart doesn't ask the question,
and the MCP server explicitly declines to answer it. **The chart is the evidence; this
is the admission.** Quote it in the framing.

### 2. Fitness knowledge is currently hand-written special cases in a system prompt

The instructions hard-code this:

> **DAAC metadata quirk:** ... Harmonized Landsat Sentinel-2 (HLS) from LPCLOUD does
> not populate the CMR `cloud_cover` field. If a `cloud_cover` filter returns 0
> granules ... advise the user to apply cloud filtering using the dataset's internal
> QA bands (e.g., the **Fmask** layer in HLS).

That is *exactly* the caveat our own recipe example encodes as a comment. NASA solved
it by writing one dataset's quirk into a prompt by hand. That does not scale past a
handful of datasets — which is the argument for the readiness record, sharpened.
Add to the memo's "PDFs and in the heads of people who have used a product for a
decade": **and in hand-written special cases in system prompts.**

### 3. Partial answer to R1 — some record fields come free, the fitness ones don't

`get_variables` returns `scale`, `offset`, `fill_values`, `valid_ranges`, `units`
from UMM-V. `get_services` returns access endpoints. So parts of **identity** and
**access** are structured and free from CMR.

**None of the fitness fields are.** No `native_resolution_m`, no
`quality_flag_convention`, no `min_meaningful_area_km2`, no `cautions[]`. Those still
have to come from ATBDs and DAAC guides — so R1 stands, but narrowed to the fields
that actually matter. Test extraction against the fitness block only; don't spend day
one re-deriving what UMM-V already gives.

## Live demonstration of the gap

Unauthenticated `get_collections`, keyword `soil moisture`, `has_granules: true`,
bbox = the memo's Boulder County polygon (-105.30, 39.90, -105.10, 40.10 — roughly
17 × 22 km). **287 hits.** Top five, in rank order:

| short_name | Grid |
|---|---|
| SPL4SMGP | 9 km |
| SPL3SMP_E | 9 km |
| SPL4SMAU | 9 km |
| SPL2SMAP_S | 3 km |
| SPL3SMP | **36 km** |

Every one of them is returned as a match for a county-scale box. The last has a single
pixel wider than the entire query polygon. The catalog says yes to all five and says
nothing about scale — the ranking is relevance, never fitness.

**This is the demo.** Real, reproducible, no auth, one call, and it fails in exactly
the way the project claims. Consider opening with it instead of a slide.
---

## Part two — live demo

Joe walks through how MCP works, highlighting where the human sits in the loop. Then
live coding, embedded in the session — slick.

Runs `/mcp`, showing servers connected at once: Drata, Datadog, Gmail, Google Calendar,
Earthmover, HubSpot, BetterStack, **earthdata**.

### The demo, and the moment it turned

Prompt: *"What repos do I have, and what is the best sea surface temperature one?"*

Claude looks up his Arraylake orgs and repos, finds **195 repos across three orgs**,
selects one as the best candidate, and explains why it chose it.

**Joe disagreed with the pick.**

The demo still worked — it showed the MCP server surfacing datasets *with context on
each* that can be used to answer the prompt. But the headline is the disagreement.

Follow-up prompt to steer it: *"I have a preference for observed SST (satellite or in
situ obs). What do I have that fits that criteria?"* — and the dataset report changes.

### Context-window discipline

> Whatever you get back from the MCP is going into the context window — so don't ask
> for 10 GB via MCP.

For bulk data, use a different tool call (curl, direct S3) and keep MCP for metadata
and control flow.

## What this means for us

### 1. An expert disagreeing with "best" is our thesis, demonstrated by someone else

The person who built the platform, holding the domain expertise, asked for the *best*
dataset and did not accept the answer. Not a strawman we constructed — the tool's own
author, on his own data, in front of the room.

Note this is the **repo** case, not even the harder Earth-science-fitness case. If
"best" fails on 195 repos you own, it fails much harder on thousands of NASA
collections you don't.

Third corroboration, now from a different direction: the chart doesn't ask the
question, NASA's MCP declines to answer it, and when an agent answers it anyway the
expert disagrees.

### 2. The fitness criterion arrived from the human, on the second prompt

*"I have a preference for observed SST (satellite or in situ obs)"* is a fitness
constraint — and Joe had to know to supply it. The agent never asked.

**That is precisely step 2 of our sequence diagram.** Our contribution is not that the
constraint can be applied; the demo shows it can. It is that the desk *elicits* it
before answering, instead of requiring the user to already know which axis matters.
Joe knew to say "observed, not model". A land manager will not know to say "30 m, not
11 km" — that is the whole point of the reference interview.

Worth adding to the gold set: an unrefined first prompt where the correct behaviour is
a question, not a ranking.

### 3. A context budget on the readiness record — a real design constraint

Joe's 10 GB warning has a direct consequence: **the record must be small enough that
several fit in context at once.** The librarian compares candidates, so it may hold
five or ten records simultaneously while ranking.

This corroborates from the NASA side too — that server's own instructions warn
"NASA Earthdata metadata is extremely verbose. Unconstrained responses can quickly
exhaust your context window," and give it a `fields` parameter and a default `limit`
of 10 for exactly this reason.

So: the readiness record is a **compact structured verdict, not a metadata dump**. If
it grows to the size of a UMM-C record we have rebuilt the problem. Two implications
for the schema — worth deciding early:

- Keep the ranking-time fields tight; put bulky provenance behind a reference rather
  than inline.
- The `{value, source, confidence}` triple is already three times the size of a bare
  value. It earns that on fitness fields where provenance decides trust. Consider
  whether it earns it everywhere, or whether identity/access fields can stay bare.
---

## Probe: does CMR already carry resolution? (2026-08-25)

`get_collections` returns `spatial_resolution` and `temporal_resolution` as
"extracted resolution strings" — so resolution *is* in the response, though it is not
a filterable parameter. (Filterable params are only: `concept_id`, `cursor`, `fields`,
`has_granules`, `instrument`, `keyword`, `limit`, `platform`, `processing_level_id`,
`provider`, `short_name`, `spatial_wkt_geometry`, `temporal_*`. No sort. No resolution
filter. No quality filter.)

Queried the four datasets from our own persona examples:

| short_name | `spatial_resolution` | `temporal_resolution` |
|---|---|---|
| HLSL30 | `"30x30 Meters"` | `"1 Day"` |
| SPL3SMP | `"36.0x36.0 Kilometers"` | `"1 Day"` |
| MOD16A2 | `"500x500 Meters"` | `"8 Day"` |
| **GPM_3IMERGHH** | **`null`** | **`null`** |

**The dataset our memo names as the trap has no resolution in the catalog at all.**
IMERG — the one the land-manager persona must be prevented from accepting — is the
one CMR does not characterise. Its 0.1° grid appears only in the free-text
`entry_title`, and converting 0.1° to ~11 km requires knowing the latitude.

The three that are populated are free-text with mixed units (`Meters` vs
`Kilometers`, `30x30` vs `36.0x36.0`) — parseable, but not comparable without
normalisation.

**This answers R1 for the most important single field.** Resolution is: partially
present, unnormalised, and null exactly where the fitness question is hardest. An
agent working from CMR alone cannot learn that IMERG is too coarse for a county,
because CMR does not say so. That is the readiness record's job, demonstrated in one
call.
---

## Slides

Stored at
[slides/2026-08-25-joe-hamman-mcp-basics.md](../slides/2026-08-25-joe-hamman-mcp-basics.md).
Source: <https://slides.earthmover.io/main/mcp-basics-hackweek/> — a Slidev SPA with no
print or PDF route, so the text was extracted from the JS bundle. 33 slides in five
parts: the problem MCP solves · how it works · unpacking calls · in practice · where it
fits, plus extras.

### Four things in the deck that bear on our design

**1. Joe states the context cost as a design rule** (slide 14):

> Tool descriptions **are** prompt engineering — the model knows nothing else, and
> **pays for the text every turn**.

And slide 31: *"more tools do not make a better agent. Each tool description fills
context on every turn. Ten sharp tools beat sixty vague ones."* Our measured ~10,900
tokens for NASA's seven tools is that rule with a number on it. Slide 27 adds
Anthropic's finding that server-side response trimming costs about **⅓ the tokens**.

**2. The readiness record is a `resource`, and the slot is empty.** Slide 13 defines
the primitive:

> A **resource** is a document that the server offers for reading: a file, a record, a
> metadata page. A URI names it. **Nothing runs when you read it.** … The user or the
> client picks it, not the model.

That is exactly the shape of a Dataset Readiness Record — a per-dataset document, named
by URI, inert on read. **NASA's `earthdata-mcp` exposes zero resources** (verified:
`resources/list` returns empty). The protocol has a slot for what we are building and
nobody has filled it. Worth deciding whether the record is served as an MCP resource
rather than only as files in our repo — it would make it consumable by any agent, which
is the D7 durability argument in a different form.

**3. Joe's three neighbours frame where each collaborator's work lives** (slide 25):
Context (`CLAUDE.md`, "how this project works") · **Skills** ("procedure — a repeatable
method") · **MCP** ("capability — the agent must reach a system"). So Aimee's readiness
assessment is a *skill* (procedure), catalog access is *MCP* (capability), and the
record itself is *neither* — it is data, which is why it wants to be a resource.

**4. Prompt injection is a named risk** (slide 32): *"the client puts tool results into
context as text. A poisoned README becomes an instruction."* Relevant to the
responsible-AI framing — if we propose that DAACs publish recipe assets that agents
read, we are proposing a new injection surface. Worth a sentence in the convention (D7)
about what a conformant `role: example` asset may and may not contain.

Also worth stealing: the **MCP Inspector** debugging rule (slide 22) — *if the tool
fails in the Inspector the fault is in your server; if it works in the Inspector but
the agent calls it wrong, the fault is in your tool descriptions.*
