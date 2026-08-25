# The slot already exists in NASA's ontology

**Research note · 2026-08-25** · verified against the CMR native API
(`collections.umm_json`) for the four persona datasets.

## Correction to an earlier claim

I previously reported that CMR carries spatial resolution as free text in mixed units
(`"30x30 Meters"`, `"36.0x36.0 Kilometers"`). **That is what the MCP server returns, not
what CMR holds.** UMM-C carries it structured:

```json
"ResolutionAndCoordinateSystem": {
  "HorizontalDataResolution": {
    "GenericResolutions": [ { "XDimension": 30, "YDimension": 30, "Unit": "Meters" } ] },
  "Description": "Universal Transverse Mercator (UTM)" }
```

The structure was there all along. The finding is not that NASA lacks it — it is that
the structure is **destroyed at the agent boundary**.

## What UMM-C actually holds, per dataset

| Field | HLSL30 | SPL3SMP | GPM_3IMERGHH | MOD16A2 |
|---|---|---|---|---|
| `ResolutionAndCoordinateSystem` | ✅ 30 m, `GenericResolutions` | ✅ 36 km, `GriddedResolutions` | ❌ **absent** | ✅ 500 m |
| `Quality.Summary` | ✅ prose | ❌ absent | ✅ prose | ❌ absent |
| `Purpose` | ❌ | ✅ | ✅ | ✅ |
| `UseConstraints` | licensing | licensing | licensing | ❌ |
| `CollectionDataType` | `SCIENCE_QUALITY` | ❌ | ❌ | `SCIENCE_QUALITY` |

And HLSL30's `Quality.Summary` opens with exactly the class of fact we want:

> For scenes greater than or equal to 80 degrees North, multiple overpasses can be
> gridded into a single MGRS tile resulting in an L30 granule with data sensed at two
> different times…

That is a **known artifact with a spatial condition** — a `cautions[]` entry in
everything but form. It is prose, so no agent can compare "≥ 80°N" against a bounding box.

## The ISO 19115 lineage — the concept is named in the standard

UMM-C descends from ISO 19115-2, and ISO names our concept outright:

| ISO element | Definition |
|---|---|
| `MD_Constraints.useLimitation` | limitations on the **"fitness of use"** of the resource — characteristics of how it was collected or processed that make it inappropriate for some specific use |
| `MD_Usage.specificUsage` | what the resource has actually been used for |
| `MD_Usage.userDeterminedLimitations` | applications for which the resource is **not suitable** |
| `MD_Usage.usageDateTime`, `userContactInfo` | when, and by whom |

ISO 19115 introduced `MD_Usage` specifically to track problems users hit with a dataset
and share them so others avoid repeating them. **That is our project, described in an
international standard NASA has adopted.**

Note the drift: **NASA's `UseConstraints` carries licensing** — `LicenseURL`,
`LicenseText` — which is the `MD_LegalConstraints` branch. The fitness branch,
`useLimitation`, has no clean UMM-C home; where it lands at all, it lands in
`Quality.Summary` as prose. The slot was defined, then colonised by legal text.

## So the gap has three distinct failure modes, not one

1. **Missing.** IMERG has no `ResolutionAndCoordinateSystem` at all. SPL3SMP and MOD16A2
   have no `Quality`. Slots defined and left empty.
2. **Unstructured.** HLSL30's `Quality.Summary` holds a machine-relevant spatial
   condition as a prose paragraph. Present but not comparable.
3. **Lost in transit.** UMM-C's `{XDimension: 30, YDimension: 30, Unit: "Meters"}`
   reaches the agent as the string `"30x30 Meters"`. Structure that exists in the
   ontology, survives to the API, and is flattened at the agent boundary.

Each needs a different fix: authoring, normalisation, and preservation.

## Where our advance slots in

**We are not proposing a new field.** We are populating, structuring, and preserving one
NASA already defined:

| Our field | Pre-existing slot | What we contribute |
|---|---|---|
| `native_resolution_m` | `UMM-C ResolutionAndCoordinateSystem.HorizontalDataResolution` → `geocr:spatialResolution` | fill where absent; preserve the structure the agent layer drops |
| `cautions[]`, `known_artifacts[]` | `UMM-C Quality.Summary` ← ISO `MD_Usage.userDeterminedLimitations` | turn prose into typed, conditioned values |
| `validated_uses[]` | `UMM-C Purpose` ← ISO `MD_Usage.specificUsage` | structure it, and separate science use from ML lifecycle |
| `min_meaningful_area_km2`, `uncertainty`, `quality_flag_convention` | **no slot in UMM-C** | genuinely new — propose as the GeoCroissant fitness extension |

Three of four are pre-existing. Only the last row is new, which makes the extension
proposal a small, well-motivated delta rather than a rival vocabulary.
