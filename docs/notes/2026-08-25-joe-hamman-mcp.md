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
