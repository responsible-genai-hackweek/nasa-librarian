# Croissant / GeoCroissant — assessment against our gap

**Research note · 2026-08-25** · prompted by Rajat pointing at Croissant.

## The headline

**Rajat Shinde (NASA IMPACT / UAH) leads GeoCroissant.** He is not referencing someone
else's standard — he is its author. GeoCroissant 1.0 was published **2026-01-29**, with
Prof. Elena Simperl (King's College London) as co-presenter on the effort.

That changes the calculus: adopting his vocabulary costs us nothing in negotiation and
buys citable lineage, and extending it is a live option rather than a hopeful proposal.

## What Croissant actually is

Not quite the knowledge-graph-to-dataset bridge it sounds like. Three layers:

| Layer | What it does |
|---|---|
| **Croissant core** | schema.org-based vocabulary describing a dataset's resources, structure and semantics, and **how to load it** into ML tools. Adopted by HuggingFace, Kaggle, OpenML, Harvard Dataverse, Google Dataset Search. |
| **Croissant-RAI** | Responsible-AI extension: collection process, annotation, limitations, biases, use cases. |
| **GeoCroissant** | Geospatial extension. Rajat's. Adds CRS, resolution, bands, temporal cadence. |

It *is* schema.org/JSON-LD, so it is linked-data shaped and Google Dataset Search
consumes it — that is the grain of truth in "knowledge graph." But its purpose is
**ML-readiness and loadability**, not concept→dataset routing.

## Field-by-field against the Dataset Readiness Record

### GeoCroissant closes the Shape block — properly

| Our field | GeoCroissant | Type |
|---|---|---|
| `crs` | `geocr:coordinateReferenceSystem` | structured, EPSG |
| `native_resolution_m` | **`geocr:spatialResolution`** | **structured `QuantitativeValue` — numeric value + unit** |
| `temporal_cadence` | `geocr:temporalResolution` | structured `QuantitativeValue` |
| — | `geocr:bandConfiguration`, `geocr:spectralBandMetadata` | structured |
| — | `geocr:spatialIndex` (H3/geohash), `geocr:timeSeriesIndex` | structured |
| endpoints | `geocr:recordEndpoint` (OGC API – Records) | URL |

`geocr:spatialResolution` is defined as *"nominal spatial resolution (ground sampling
distance) represented by one pixel on the ground"* — **as a number with a unit.** That
is exactly the normalised field CMR does not have. Our probe found CMR returning
`"36.0x36.0 Kilometers"` and `"30x30 Meters"` as free text, and `null` for IMERG.

### It does **not** close the Fitness block — the half the project is about

| Our field | GeoCroissant / RAI | Gap |
|---|---|---|
| `quality_flag_convention` | — | **absent.** The HLS Fmask case has no home |
| `uncertainty` | — | **absent** |
| `min_meaningful_area_km2` | — | **absent** |
| `known_artifacts[]` | — | absent |
| `cautions[]` | `rai:dataLimitations`, `geocr:spatialBias` | free text, and aimed elsewhere |
| `validated_uses[]` | `rai:dataUseCases` | enum: Training / Testing / Validation / Fine Tuning — **ML lifecycle, not science use** |

And the whole **Recipe & verdict** block — `access_recipe`, `recipe_last_verified`,
`cloud_ready`, `ai_ready`, `blockers[]` — has no counterpart at all.

Documented escape hatch for what is missing: `sc:additionalProperty` with
`sc:PropertyValue`, or external vocabulary mappings in the JSON-LD `@context`.

### The orientation mismatch, stated plainly

GeoCroissant's RAI fields are about **model generalisation** — `geocr:spatialBias` means
"this training set under-samples the tropics." Our question is **fitness at the scale
asked** — "9 km cannot separate this farmer's fields." Related, not the same. Do not
assume the RAI block already covers us; it does not.

## The trap that has not gone away

**Croissant is a vocabulary, not a corpus.** `geocr:spatialResolution` existing does not
mean any NASA collection has a populated value. IMERG's `null` in CMR does not become
11 km because a field for it is defined somewhere.

Somebody still has to produce the values. **That is Aimee's job and it is unchanged.**
Same shape as the ESIP trap, one level up: schema ≠ records.

## Verdict

**Adopt GeoCroissant as the record's vocabulary. Do not invent our own field names.**

- Rajat authors it — authoritative and free
- Published, versioned, dated (1.0, 2026-01-29)
- Normalises resolution as a quantity, which is precisely what CMR lacks
- Citable lineage, the same argument that made us reach for ESIP

**And propose the fitness block as a GeoCroissant extension.** That is a much stronger
D7 than a from-scratch STAC convention: there is a published spec, a working group, and
our collaborator leads it. Candidate extension fields: `min_meaningful_area_km2`,
`quality_flag_convention`, `uncertainty`, `known_artifacts[]`, and the verdict triple.

## Adjacent: there is already a GeoCroissant MCP server

[`HarshShinde0/geocr_mcp`](https://github.com/HarshShinde0/geocr_mcp) — created
2026-08-23, **pushed 2026-08-25 05:02 UTC, hours before this note.** No license
declared. Also [`HarshShinde0/geocroissant`](https://github.com/HarshShinde0/geocroissant)
(Python library, adds bounding boxes, CRSs, band configs, spatial resolution, raster ops)
and a QGIS plugin.

Its tools: `list_eo_catalogs`, `search_eo_datasets`, `get_eo_dataset_details`,
`geocode_place`, `count_eo_scenes`, `create_geocroissant_from_stac`,
`create_geocroissant_from_stac_sources`, `create_geocroissant_scaffold`,
`validate_croissant`, `inspect_geocroissant`, `ping`.

**Read the orientation carefully: this is a *producer* tool.** It generates GeoCroissant
JSON-LD *from* STAC searches and validates it. It does not serve fitness verdicts, and
it does not rank. It is plausibly the machine that mass-produces Aimee's records —
which is a large gift — but it is not the librarian and does not overlap the reference
desk.

Ask Rajat directly: is `geocr_mcp` the intended production path, and what populates
`geocr:spatialResolution` when the source STAC collection does not carry it?

## MCP Inspector

Both Joe and Jason demoed the **MCP Inspector** (`npx @modelcontextprotocol/inspector`)
as the way to make a server explicit — a browser UI speaking MCP directly, no model, no
agent. Joe's rule (slide 22):

> If the tool fails in the Inspector, the fault is in your server. If the tool works in
> the Inspector but the agent calls it wrong, the fault is in your **tool descriptions**.

And slide 23: opening a tool in the Inspector shows *"this text is all that the model
sees."*

**Action:** run the Inspector against `cmr.earthdata.nasa.gov/mcp/v1` and against
`geocr_mcp`. It is the fastest way to judge whether either is usable for our flow, and
it reads the same text the librarian would be reasoning from.
