# Decision Log

## Open decisions

Seven decisions and one open risk, carried over from the architecture memo of
2026-08-24. Ordered by when they stop being free to answer — the first three are gated
on nothing and block everything.

Status values: `open` · `leaning` · `decided` · `superseded`.
When one is settled, move it down to **Decided** below with context and consequences.

### Before anyone codes

#### D1 — Which region, question and persona?
**Status:** open · **needs an owner**

Fixtures, gold set and demo script all descend from this, and none can start without
it. Owner should be whoever owns the science framing. Assign this first, not last.

#### D2 — Which providers may the librarian search?
**Status:** leaning — three

CMR-STAC is partitioned per DAAC; there is no global granule search, so the shelf is an
explicit list rather than a wildcard. Wide looks good on a slide and does not finish in
a weekend.

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
**None of the fitness fields do** — `native_resolution_m`, `quality_flag_convention`,
`min_meaningful_area_km2` and `cautions[]` are still ATBD/DAAC-guide territory. Test
extraction against the fitness block only; don't spend day one re-deriving UMM-V.

---

## Decided

*(none yet — move entries here from Open decisions as they settle)*

---

## Template for new decisions

## [YYYY-MM-DD] - Decision Title

### Context
What is the issue that we're seeing that is motivating this decision?

### Decision
What is the change that we're proposing?

### Consequences
What becomes easier or more difficult to do because of this change?
