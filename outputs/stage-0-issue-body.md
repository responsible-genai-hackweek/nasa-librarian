# Stage 0: Refine and Enrich User Stories Following an Information Architecture

## Objective

Transform thin, tacit user stories into explicit, structured specifications that an LLM + earthdata-mcp can reason about. The goal is to make the implicit domain knowledge **auditable and testable**.

## Information Architecture: Template for an Enriched User Story

Each refined user story should capture these elements:

### 1. Persona Context
- **Role & Organization:** What do they do, and where/for whom?
- **Expertise Level:** Domain expert? Technical? Non-technical?
- **Stakes:** Why does this matter? (Policy, safety, operations, research?)

### 2. Raw Problem Statement
- **In their words:** What's the surface problem?
- **Root need:** What are they actually trying to decide or understand?

### 3. The Six Slots (Explicit Specification)

#### Unit of Analysis
- *What* is the thing that must be resolved?
- Examples: one parcel (40 ha), a county (2000 km²), a watershed, a bridge, a specific structure
- **Why it matters:** Determines whether resolution is adequate

#### Time Window & Horizon
- **Retrospective window:** How far back must we look? (5 years of trend, 30 years, since monitoring began?)
- **Forward horizon:** Do they need prediction, or just understanding of the past?
- **Why it matters:** Determines whether record length is adequate

#### Cadence Required
- **Update frequency:** Daily? Weekly? Seasonal? Annual? Once?
- **Why it matters:** Determines whether temporal cadence of data fits operational needs

#### Variable Meaning
- **The phenomenon they're actually measuring:** What does "drying" mean here? (soil moisture? vegetation stress? precipitation deficit? streamflow?)
- **Which measurement layer?** (surface, root-zone, canopy?)
- **Physical units:** Percentage? mm? Index (0-100)?
- **Why it matters:** Determines semantic match to dataset

#### Effect Size (Detectability Threshold)
- **What change would matter?** (±10% of baseline? ±1σ? Crossing a threshold?)
- **Why it matters:** Determines whether measurement uncertainty is acceptable

#### Latency Tolerance
- **Operational (real-time needs):** Answer must come within hours/days; drives decision now
- **Retrospective (analytical):** Answer can come weeks/months later; understanding past
- **Why it matters:** Determines whether data latency is acceptable

### 4. Gotchas & Traps
- **What could mislead a naive picker?** 
- **Regional caveats:** (e.g., SMAP fails over dense vegetation; this region is half-forested)
- **Measurement artifacts:** (e.g., ice clouds mask snow; thermal inertia lags air temp)
- **What would the wrong answer look like?**

### 5. Dataset Suitability Criteria
- **Would this work?** Example dataset and why
- **Wouldn't this work?** Example dataset and why

---

## The Work: Enriched User Stories

Below, each raw user story has been refined and enriched following this template. These are draft interpretations; the persona can correct or clarify.

---

### Story 1: Beavers in the Arctic

**Persona Context**
- Infectious disease researcher / public health official
- Expertise: epidemiology, public health policy (not necessarily remote sensing)
- Organization: likely public health agency or academic institution
- Stakes: Disease vector control, permafrost stability, infrastructure protection

**Raw Problem**
"Beavers are moving into subarctic regions, introducing infectious disease, and accelerating permafrost thaw, which threatens infrastructure and releases greenhouse gases."

**Need Statement**

| Dimension | Specification |
|---|---|
| **Unit of Analysis** | Subarctic region of interest (likely North Slope Borough or regional extent, 100s-1000s km²); sub-region habitat patches where beavers could establish |
| **Time Window** | 10–20 year retrospective (when did beaver presence increase?); forward: 5–10 year projection of permafrost state |
| **Cadence** | Annual or seasonal (captures seasonal patterns of activity, meltwater, vegetation); not sub-seasonal |
| **Variable Meaning** | (A) Beaver presence: vegetation change from flooding/damming (localized devegetation or shrub expansion). (B) Permafrost state: ground temperature, active-layer thickness, subsidence; not direct ice content |
| **Effect Size** | Vegetation change ≥5–10% in a patch (detectable by satellite); permafrost subsidence ≥1 cm/year (measurable); temperature rise ≥0.1°C/year (significant trend) |
| **Latency** | Retrospective: seasonal reporting acceptable; not operational (not a real-time response trigger) |

**Gotchas & Traps**
- **Regional trap:** Beaver dams cause localized flooding → spectral signature changes (green increase, then browning). Not the same as regional drought.
- **Permafrost trap:** Ground temperature is sparse; active-layer thickness requires in-situ or InSAR; subsidence is visible in InSAR but requires multi-year stacks.
- **Seasonal trap:** Winter makes everything white; active season (June–Sept) is when beavers are visible.
- **Trap for naive picker:** Taking "permafrost thaw" → searching "temperature" → getting air temperature (not ground temp) or coarse global models (not regional detail).

**Would Work**
- Landsat / Sentinel-2 time series (30 m) for vegetation change detection over 10+ years
- InSAR (SAR displacement) for subsidence if available at 10+ m resolution over 5+ years
- Regional permafrost models (NSIDC, NCAR) for ground temp and ALT

**Wouldn't Work**
- SMAP (36 km): too coarse to detect a beaver-dam patch
- IMERG (11 km): precipitation proxy, not permafrost state
- Microwave backscatter alone: cannot separate water from organic soil
- < 5-year time series: cannot establish trend

---

### Story 2: North Slope Borough Search and Rescue

**Persona Context**
- Search and Rescue manager
- Organization: North Slope Borough emergency response
- Expertise: operations, local geography, weather patterns (not remote sensing)
- Stakes: Safety of people at risk; response time critical

**Raw Problem**
"A SAR manager in Utqiagvik must predict and respond to coastal hazards from weather, sea ice motion, and related atmospheric/oceanographic variables."

**Need Statement**

| Dimension | Specification |
|---|---|
| **Unit of Analysis** | Coastal region of Utqiagvik and immediate offshore (10–50 km, depending on search scope) |
| **Time Window** | Forecast: 3–7 days ahead (tactical decision window for SAR deployment). Climatological context: 10-year history of ice breakup timing, storm frequency |
| **Cadence** | Daily or sub-daily for operational decisions; seasonal for planning |
| **Variable Meaning** | (A) Sea ice motion: pack ice velocity, concentration, edge position. (B) Weather: wind speed/direction, atmospheric pressure, precipitation. (C) Coastal hazards: inundation risk (surge + swell). Not general weather, but ocean-atmosphere coupling. |
| **Effect Size** | Ice motion ≥10 cm/s (notable drift); wind ≥15 m/s (hazardous); sea level rise ≥0.5 m above normal (inundation risk) |
| **Latency** | Operational: forecast valid 3–7 days out; latency must be < 6 hours from observation to product |

**Gotchas & Traps**
- **Operational trap:** A non-operational dataset (e.g., weekly composite, 3-month latency) is useless for SAR response.
- **Scale trap:** Coastal hazards are mesoscale (10–100 km); too-coarse data (global models, 100+ km) misses local channeling, coastal jets.
- **Sea ice trap:** Concentration alone is not enough; you need motion/drift to predict where ice will be.
- **Trap for naive picker:** "Weather forecast" → general weather model (coarse, not coastal). Need coupled ocean-atmosphere regional model or satellite sea-ice motion.

**Would Work**
- Sea ice motion from SAR or passive microwave (daily, ~10 km scale) over coastal zone
- High-resolution regional weather model (e.g., NOAA GFS at 0.25°, or nest down to 5 km)
- Sentinel-1 sea ice edge and motion (12-day revisit, 10–100 m resolution)
- Coastal inundation models with real-time forcing

**Wouldn't Work**
- MODIS (2000 m, 1–2 day latency): too coarse for coastal detail, latency too long
- Monthly or seasonal products: no tactical forecast window
- Global ice concentration: no motion/drift info
- Historical climate model: not real-time forecast

---

### Story 3: What's in the Water? (Algal Blooms & Fjord Productivity)

**Persona Context**
- Environmental scientist or fisheries manager
- Organization: fishery agency, coastal research institute, or NGO
- Expertise: oceanography, limnology (specialized domain knowledge)
- Stakes: Fishery productivity, ecosystem health, public health (harmful algal blooms)

**Raw Problem**
"Distinguish the sources of water-color changes: turbidity vs. chlorophyll vs. harmful algal blooms. Also assess fjord productivity in the presence of glacial melt, complex currents, and sediment."

**Need Statement**

| Dimension | Specification |
|---|---|
| **Unit of Analysis** | Fjord or coastal embayment (10–100 km scale); individual bloom patch (1–10 km) for detection |
| **Time Window** | Seasonal (ice-out to freeze-up, ~4–6 months); retrospective 5–10 years to establish baseline productivity trends |
| **Cadence** | 2–5 days during productive season (near-daily observations); weekly acceptable for trend |
| **Variable Meaning** | Spectral reflectance decomposed into: (A) chlorophyll concentration (proxy for phytoplankton). (B) Suspended sediment (turbidity). (C) Colored dissolved organic matter (CDOM). (D) Harmful algal bloom species (if spectral signature distinct). For fjord productivity: net primary production or chlorophyll as proxy. |
| **Effect Size** | Bloom detection threshold ≥10 mg/m³ chlorophyll (typical nuisance threshold); productivity change ≥10–20% from baseline; sediment plume visible (> 1 mg/L visible in coastal zone) |
| **Latency** | Near real-time for bloom alert (1–2 days); retrospective analysis acceptable for productivity trends |

**Gotchas & Traps**
- **Spectral trap:** Fjord sediment plumes can mimic bloom color if only RGB/natural-color viewed. Need multispectral (at least 4–5 bands in blue-red).
- **Glacier trap:** Glacial meltwater is highly turbid (high sediment). Without separating sediment from pigment, true bloom goes undetected.
- **Cloud trap:** High-latitude and maritime: frequent cloud cover can make daily sampling impossible; 5-day composite may blur rapid bloom dynamics.
- **Harmful algal bloom trap:** Not all high-chlorophyll is dangerous; species detection requires specific spectral or in-situ confirmation. Satellite cannot definitively ID species without ground truth.
- **Trap for naive picker:** "Algal bloom" → "green water" → any green-pixel detector. Need reflectance decomposition to isolate chlorophyll from sediment and CDOM.

**Would Work**
- Sentinel-2 / Landsat-8/9 (10–30 m, 3–5 day revisit, multispectral, red edge band): excellent for coastal spectral decomposition
- MODIS (500–1000 m, daily): coarser but global daily coverage; gap-filler for clouds
- Copernicus Sentinel-3 OLCI (300 m, daily): high temporal resolution, designed for coastal color
- Regional ocean models with primary production parameterization (forecasts NPP)

**Wouldn't Work**
- Coarse thermal data (MODIS LST, 1 km): cannot resolve fjord detail
- Passive microwave (25+ km): completely misses fjord-scale features
- Radar backscatter alone: no pigment information
- Weekly or coarser cadence: misses rapid bloom development or dissipation
- RGB/natural-color only: cannot decompose sediment from algae

---

### Story 4: Where Will the Ground Collapse? (Permafrost Subsidence)

**Persona Context**
- Geotechnical engineer or infrastructure planner
- Organization: municipality, DOT, or federal land agency (Alaska, Canada, Russia)
- Expertise: civil engineering, foundation design (not necessarily remote sensing)
- Stakes: Infrastructure safety, cost of mitigation, planning retreat vs. adapt

**Raw Problem**
"Identify regions of high permafrost ice richness and subsidence risk to avoid infrastructure siting or plan for adaptation."

**Need Statement**

| Dimension | Specification |
|---|---|
| **Unit of Analysis** | Community or infrastructure corridor (1–10 km); also landscape-scale assessment (100+ km²) for regional planning |
| **Time Window** | Historical (30+ years to establish subsidence trends); forward (10–20 year projection of thaw) |
| **Cadence** | Annual or inter-annual for subsidence monitoring; seasonal for active-layer thickness |
| **Variable Meaning** | Ice richness (fraction of pore space filled with ice; %; mapped from cores or proxy). Active-layer thickness (depth to permafrost; m). Subsidence rate (vertical displacement; mm/year to cm/year). Ground temperature (°C). Not just permafrost presence (binary), but quantitative state. |
| **Effect Size** | Subsidence ≥1–5 mm/year (becomes structurally significant over decades); ALT change ≥5–10 cm/year (indicates rapid thaw); ground temp rise ≥0.1–0.2°C/year (significant trend) |
| **Latency** | Retrospective (planning window is years); not operational |

**Gotchas & Traps**
- **Ice richness trap:** In-situ only (boreholes, cores). Satellite cannot directly measure. Proxies exist (vegetation type, EVI, but these are indirect).
- **Subsidence trap:** Requires multi-year SAR (InSAR or PSI). Sparse Arctic coverage; many regions have no InSAR history. Vertical resolution limited to ±5 mm, but full stack required (not single pair).
- **Ground temperature trap:** In-situ permafrost stations sparse. Regional models exist but coarse (10–100 km). Not satellite-derived.
- **Trap for naive picker:** "Permafrost" → "Temperature" → air temp or coarse global model. Need ground temp or subsidence proxy or active-layer thickness.
- **Artifact trap:** InSAR can be fooled by snow, water, seasonal deformation. Multi-seasonal stacking required.

**Would Work**
- InSAR (SAR-derived subsidence) over 10+ years at 10–100 m resolution (Sentinel-1, ALOS2)
- Permafrost temperature models (NSIDC, UiO) at 1 km scale
- Active-layer thickness from satellite-derived phenology + reanalysis (SMAP indirect)
- Vegetation indices as permafrost-stability proxy (EVI, NDVI over 20+ years)
- Drill-core data from DGGS, USGS (localized but ground truth)

**Wouldn't Work**
- Passive microwave alone (SMAP, AMSR): cannot resolve sub-regional heterogeneity
- Coarse models (global GCM, 100+ km): miss local variability
- Single-year snapshot: subsidence requires multi-year trend
- SAR single-pair: only useful over stable glaciers; permafrost needs full stack

---

### Story 5: Where's the Rust? (Rusting Rivers in Mining Regions)

**Persona Context**
- Environmental scientist or mining-impact monitor
- Organization: EPA, state/tribal agency, environmental advocacy org
- Expertise: hydrochemistry, acid mine drainage (specialized domain)
- Stakes: Water quality, ecosystem health, remediation planning

**Raw Problem**
"Monitor changes in redness (Fe oxidation) in rivers and streams downstream of mining operations. Predict recovery timeline given oxidizable resource depletion."

**Need Statement**

| Dimension | Specification |
|---|---|
| **Unit of Analysis** | Stream reach or drainage (10–100 km); mining-affected tributaries (1–10 km) where color change is visible |
| **Time Window** | Retrospective 10–20 years (to establish baseline and change). Forward 5–10 years (resource depletion timeline assumption). |
| **Cadence** | Seasonal (high-flow vs. low-flow affects visibility and discharge). ~Monthly or quarterly for trend. |
| **Variable Meaning** | Iron oxidation state (Fe²⁺ → Fe³⁺): mapped as spectral hue shift (reddish-yellow to brownish-red). Proxy: hue angle in HSV color space or specific band ratios (e.g., red/blue). Also water discharge and suspended sediment concentration. |
| **Effect Size** | Detectable hue shift (Δhue ≥10–20° in HSV); concentration ≥10 mg Fe/L visible; sediment plume extending >5 km downstream |
| **Latency** | Retrospective; seasonal reporting acceptable. Not operational. |

**Gotchas & Traps**
- **Color trap:** "Redness" is subjective. Must define in terms of spectral reflectance (hue, specific band ratios). Natural reddish soils can confound visual interpretation.
- **Scale trap:** Mining-impacted streams are often small (< 10 m wide); satellite pixels (10–30 m) may not resolve. Drone or airborne imaging needed for detailed reach-scale assessment.
- **Seasonal trap:** High flow (spring melt, storms) dilutes color and increases turbidity; low flow (summer, winter) concentrates color and clarifies. Comparing across seasons is problematic.
- **Saturation trap:** If Fe precipitate is already on banks and in sediment (years of AMD), stream hue stabilizes; change becomes hard to detect even as mass flux continues.
- **Trap for naive picker:** "Redness" → any red-channel pixel. Need validated ratio or hue metric correlated to Fe concentration.

**Would Work**
- Sentinel-2 / Landsat (10–30 m, multispectral, red edge): can resolve small rivers, measure hue and spectral ratios
- Landsat archive (30+ years of 30 m data): establish long-term trend
- Sentinel-1 SAR (10 m, penetrates some cloud): gap-filler in cloudy regions
- Drone / airborne visible (10 cm–1 m, tactical surveys for ground truth and detailed mapping)
- USGS water-quality monitoring stations (ground truth for spectral calibration)

**Wouldn't Work**
- MODIS (1000 m): too coarse to resolve streams < 1 km
- Passive microwave: no visible reflectance info
- Annual or coarser cadence: seasonal patterns confound trend detection
- SAR alone (no color info)
- Single snapshot: trend requires multi-year time series

---

### Story 6: Are the Trees Alive? (Tree Mortality Detection)

**Persona Context**
- Forest ecologist or land manager
- Organization: Forest Service, state forestry, academic research
- Expertise: forest dynamics, dendroecology (specialized domain)
- Stakes: Pest/disease management, carbon inventory, fire risk assessment

**Raw Problem**
"Map tree mortality in near-real time using high-resolution satellite with high temporal resolution and infrared bands (to distinguish live vs. dead foliage)."

**Need Statement**

| Dimension | Specification |
|---|---|
| **Unit of Analysis** | Stand or forest patch (100 m–10 km, depending on tree density and management unit). Pixel-scale: ≥30 m (tree crown scale for boreal/northern forests). |
| **Time Window** | Seasonal (growing season = tree phenology). Retrospective 10+ years (long-term mortality trend). Forward 1–3 years (prediction of outbreak spread). |
| **Cadence** | 2–5 days during growing season (June–September); weekly acceptable for slower processes. |
| **Variable Meaning** | Vegetation stress: (A) NDVI decline (green decrease from chlorophyll loss). (B) NIR/SWIR shift (dead wood has different reflectance than live leaves). (C) Specific indices: NDMI (moisture stress), EVI (foliage density). Not just "any vegetation," but specifically tree crowns in conifer forests. |
| **Effect Size** | Detectable tree death ≥5–10% mortality in a stand (crown closure change, NDVI drop ≥0.1). Stress progression: NDVI decline ≥0.05/year indicates rapid decline. |
| **Latency** | Near-real-time for outbreak alerts (1–2 weeks lag acceptable); retrospective for long-term assessment. |

**Gotchas & Traps**
- **Deciduous trap:** Deciduous trees lose leaves seasonally (natural); mistaken for mortality. Must distinguish phenology from mortality. Requires year-to-year comparison at same calendar date.
- **Scale trap:** Small-scale mortality (scattered trees) is invisible at 30 m resolution; needs ≤10 m for individual tree detection. Landsat (30 m) shows only > 1-hectare patches.
- **Spectral trap:** Brown dead trees look similar to brown/reddish vegetation stress (drought). Need multi-year context and phenology model to distinguish.
- **Cloud trap:** High-latitude boreal forest: frequent cloud cover; 5-day revisit may not yield clear imagery.
- **Trap for naive picker:** "Tree mortality" → "NDVI decrease" → any green-to-brown pixel (including grassland senescence, crop harvest).

**Would Work**
- Sentinel-2 (10–20 m, 5-day revisit, multispectral with NIR/SWIR, red-edge): excellent temporal resolution and spectral detail
- Landsat-8/9 (30 m, 8-day revisit, multispectral NIR/SWIR): long archive, good for trend
- PlanetLabs SkySat or Maxar (1–3 m, daily): very high resolution but expensive and limited area
- Phenology models (e.g., TIMESAT, seasonal NDVI curves) to establish baseline and detect anomalies
- MODIS 250 m (2-day revisit): gap-filler for clouds
- LiDAR (if available from NASA's GEDI): 3D tree structure, but global coverage is sparse

**Wouldn't Work**
- MODIS 500 m or coarser: cannot resolve individual stands
- Thermal data alone: no vegetation index
- SAR alone: forest phenology not visible in radar
- Single snapshot: phenology requires temporal series
- Monthly or coarser cadence: misses rapid decline from outbreak

---

### Story 7: Glaciohydrology (Hydropower Management)

**Persona Context**
- Hydropower plant operator or water resources manager
- Organization: utility company, state water agency
- Expertise: hydrology, reservoir operations (not remote sensing)
- Stakes: Consistent power generation, water supply, downstream flow commitments

**Raw Problem**
"Use satellite data to forecast meltwater contribution to reservoirs so that operators can manage flow consistency and plan operations."

**Need Statement**

| Dimension | Specification |
|---|---|
| **Unit of Analysis** | Glacier extent and area (100–1000 km²); watershed draining to reservoir (100–10000 km²); specific glaciers contributing to a reservoir |
| **Time Window** | Seasonal (spring snowmelt through late summer). Forecast 1–3 months ahead. Retrospective 20–30 years (climate trend, glacier retreat). |
| **Cadence** | Weekly or bi-weekly for melt-season operations; monthly for long-term trend. |
| **Variable Meaning** | Snow/ice area (km²), snow-line elevation (m asl), glacier surface mass balance (mm water equivalent), meltwater discharge (m³/s). Not just ice presence (binary), but quantitative extent and melt rate. |
| **Effect Size** | Snow-line movement ≥100 m (indicates intensifying melt). Glacier-area loss ≥1–5% per year (long-term retreat). Runoff anomaly ≥20% from climatological mean. |
| **Latency** | Short-term forecast (1–3 months): latency < 1 week. Long-term planning (decade scale): latency months. |

**Gotchas & Traps**
- **Firn trap:** Old snow (firn) and bare glacier ice can have similar spectral signatures; NDVI alone cannot separate. Need multispectral decomposition.
- **Cloud trap:** Alpine and coastal mountains: frequent orographic clouds. Single-date revisit useless; 5–10 day composite needed.
- **Seasonal trap:** Snow/ice spectral signature changes with age and melt. Early season (new snow, bright) vs. late season (bare ice, dark). Same glacier has very different NDVI.
- **Accumulation/ablation trap:** Satellite sees surface; cannot directly measure subsurface mass balance or internal firn changes.
- **Trap for naive picker:** "Glacier extent" → "white pixel" → clouds, avalanche deposits, persistent snow confound. Need time-series classification (cloud persistence filtering).

**Would Work**
- Sentinel-2 (10–20 m, 5-day revisit, multispectral): excellent for snow-line mapping
- Landsat-8/9 (30 m, 8-day revisit): long archive for trend
- MODIS (500 m, daily): fills clouds and provides continuous monitoring
- Radar interferometry (Sentinel-1 SAR for glacier motion, though not direct melt)
- Hydrological models forced with satellite snow/ice data (e.g., glacio-hydrological models)
- Streamflow gauges (ground truth) to calibrate satellite-derived discharge estimates

**Wouldn't Work**
- Thermal data alone (no snow classification)
- Coarse passive microwave (SMAP, AMSR, 25+ km): cannot resolve glacier-scale detail
- Weekly or coarser cadence: melt can change rapidly over days
- Single band (e.g., NIR only): cannot robustly separate snow from cloud

---

### Story 8: Air Traffic Control in Active Volcanic Areas

**Persona Context**
- Air traffic controller or aviation safety official
- Organization: FAA, ICAO, regional airport authority
- Expertise: aviation operations, not volcanology or remote sensing
- Stakes: Aircraft safety, flight disruption, hazard avoidance

**Raw Problem**
"Detect volcanic ash plumes in near-real time to issue airspace closures and reroute traffic around ash hazards."

**Need Statement**

| Dimension | Specification |
|---|---|
| **Unit of Analysis** | Airspace around active volcano (100–1000 km radius, depending on plume transport); individual ash plume (10–100 km horizontal extent) |
| **Time Window** | Real-time (minutes to hours) for plume detection and nowcast. Short-term forecast (6–24 hours for plume trajectory). |
| **Cadence** | As frequent as possible; ideally 15–60 min updates during eruptive activity. |
| **Variable Meaning** | Volcanic ash plume: presence (yes/no), concentration (mg/m³ or optical depth), altitude (m asl or FL flight level), trajectory (direction and speed). Not all white clouds are ash; must discriminate. |
| **Effect Size** | Detectable ash concentration ≥0.2 mg/m³ (hazard threshold for engines). Plume altitude ≥flight level of concern (depends on local airspace). |
| **Latency** | Operational: latency < 30 minutes from observation to alert; < 6 hours for trajectory forecast |

**Gotchas & Traps**
- **Cloud trap:** Ash plumes and water clouds look similar in visible imagery. Must use thermal and multispectral signatures (ash has different temperature and emissivity than water cloud).
- **Small plume trap:** Young ash plume may be < 10 km wide; satellite pixels (1–4 km at best for thermal) may miss. Requires geostationary data (15-min revisit, but coarser resolution).
- **Altitude trap:** Satellite cannot directly measure altitude; must infer from dispersion, wind field, and thermal signature.
- **Forecast trap:** Plume trajectory depends on upper-level winds, which are uncertain. Forecasts > 12 hours have large error.
- **Trap for naive picker:** "Volcanic cloud" → any high cloud detected. Need ash-specific multispectral algorithms (11 µm thermal + 10.3 µm ash band, available on some satellites).

**Would Work**
- Geostationary satellites (GOES, HIMAWARI) with thermal + multispectral: 15–60 min revisit, dedicated ash detection algorithms
- Sentinel-5P / Tropomi (CO, SO₂ absorption): identifies volcanic signature
- MODIS (1–2 km thermal, 4–6 times daily): regional ash characterization
- Volcanic Ash Advisory Centers (VAAC) forecasts forced with satellite ash observations
- Ground-based lidar and ceilometer at key airports (ash layer altitude)

**Wouldn't Work**
- Polar-orbiting thermal only (1–2 passes/day, 1–4 hour latency): misses plume evolution
- Passive visible only: cannot separate ash from water cloud
- Non-thermal multispectral (< 11 µm): ash signature requires thermal bands
- Coarse resolution (> 10 km): small plumes undetectable
- No forecast model: satellite gives now-cast only; trajectory uncertain

---

### Story 9: Is This Bridge Going to Collapse? (Structural Monitoring)

**Persona Context**
- Civil engineer or infrastructure owner
- Organization: state DOT, bridge authority, municipal government
- Expertise: civil engineering, structural mechanics (not remote sensing)
- Stakes: Public safety, costly repairs, traffic disruption

**Raw Problem**
"An aging bridge suspected of movement near a fault line needs monitoring. Determine if it is moving and at what rate."

**Need Statement**

| Dimension | Specification |
|---|---|
| **Unit of Analysis** | Individual bridge structure (10–1000 m span); also surrounding area (> 1 km) to establish reference points free from fault motion |
| **Time Window** | Long-term (years to decades) to establish baseline and separate secular trend from seismic/thermal cycles |
| **Cadence** | Regular monitoring (weekly to monthly updates); not continuous. |
| **Variable Meaning** | Vertical and/or horizontal displacement (mm to cm). Differential movement (one span sinking relative to another). Crack propagation (not visible from satellite). **Not** just proximity to fault; need *actual displacement*. |
| **Effect Size** | Detectable movement ≥5–10 mm/year (structural concern threshold); differential ≥1–5 mm over span (bearing stress indicator). |
| **Latency** | Retrospective (long-term trend); not operational. |

**Gotchas & Traps**
- **Scale trap:** Bridge is a few meters to km; SAR pixels are 10–100 m. Must stack many pixels and use InSAR coherence to extract sub-pixel motion. **Unlikely to work for a single bridge; better for bridge array or dam.**
- **Reference trap:** Must have stable reference points far from faults to subtract regional deformation. Urban areas often have non-ideal reference pixels.
- **Thermal trap:** Concrete expands/contracts with temperature; seasonal signal (mm scale) often exceeds tectonic signal. Need multi-year stack and seasonal filtering.
- **Atmosphere trap:** Water vapor in atmosphere adds noise; interferometric processing difficult over water or dense vegetation.
- **Coherence trap:** Bridge structure may decorrelate (changing materials, traffic vibration, vegetation); cannot maintain coherence over long intervals.
- **Trap for naive picker:** "Bridge movement" → SAR + InSAR → assume sub-cm accuracy. In practice, ±5 mm standard error is optimistic; over 1–5 years, tectonic signal ~ thermal noise.

**Would Work**
- Persistent scatterer (PS) InSAR or Small Baseline (SBAS) approach over 10+ year Sentinel-1 stack: identifies stable pixels on bridge and nearby
- High-rate GPS (continuous or campaign) at bridge abutments: gold standard, mm-level accuracy
- Ground-based radar or laser (inclinometers, LVDTs): direct measurement of movement
- SAR for identifying liquefaction or ground failure nearby (indirect evidence)

**Wouldn't Work**
- Single-pair SAR interferogram: requires stacks for mm-level precision
- Optical satellite (InSAR not applicable)
- Coarse thermal: cannot resolve individual structures
- Satellite radar alone over water or vegetation: coherence lost
- < 5 year time series: thermal noise exceeds tectonic signal

---

### Story 10: LA Olympics 2028 — Multi-Hazard Risk Assessment

**Persona Context**
- Urban planner, public health official, emergency manager
- Organization: LA City or County, Olympic Committee, emergency management
- Expertise: urban planning, public health, emergency response (not remote sensing)
- Stakes: Public safety, event security, operational planning

**Raw Problem**
"Identify which parts of LA are at highest risk for heat stress, poor air quality, seismic activity, and wildfires during the 2028 Olympics."

**Need Statement**

This breaks into four sub-needs (each with its own dataset profile):

#### 10a. Heat Stress Risk

| Dimension | Specification |
|---|---|
| **Unit of Analysis** | City blocks to neighborhoods (100 m–1 km); Olympic venue sites (point locations) |
| **Time Window** | Historical summer climatology (20+ years, June–September). Forecast summer 2028 (3-month ahead). |
| **Cadence** | Daily during event (real-time heat index). Weekly for planning forecast. |
| **Variable Meaning** | Land surface temperature (°C, spatially resolved). Also air temperature, humidity, wind. Combine into heat index. Urban heat island effect (LST higher in built areas). |
| **Effect Size** | Heat island amplitude ≥3–5°C above surrounding rural. Heat index > 38°C (dangerous threshold). |
| **Latency** | Real-time (1-day latency acceptable) for event operations. |

**Would Work**
- Landsat 8/9 thermal (100 m, 8-day revisit, LST product)
- MODIS LST (1 km, daily)
- Copernicus Sentinel-5P temperature (coarse)
- Urban temperature surveys + modeling (e.g., VEGA model)

**Wouldn't Work**
- Air temperature point stations only: misses spatial heterogeneity
- Cloud-contaminated thermal: gaps during overcast days
- Coarse forecast model (100+ km grid): misses neighborhood-scale variation

#### 10b. Air Quality (AQI / Ozone)

| Dimension | Specification |
|---|---|
| **Unit of Analysis** | Air quality basin (100–1000 km²); Olympic venue sites |
| **Time Window** | Historical (10+ years, summer baseline). Forecast summer 2028 (seasonal). |
| **Cadence** | Daily during event. |
| **Variable Meaning** | NO₂, O₃, PM2.5 concentration (µg/m³). Also AQI index (composite metric). |
| **Effect Size** | AQI > 150 (unhealthy; athletic performance impaired). O₃ > 70 ppb (sensitive groups at risk). |
| **Latency** | Forecast 3–5 days ahead. Real-time observations 1-day latency. |

**Would Work**
- Sentinel-5P / Tropomi NO₂ & O₃ (multispectral satellite)
- EPA air-quality monitoring stations (ground truth)
- NOAA/EPA air-quality forecasts (chemical transport models)
- MODIS aerosol optical depth (proxy for PM)

**Wouldn't Work**
- Satellite alone (coarse, delayed): must blend with models and station data
- Forecasts > 1 week: uncertainty grows rapidly

#### 10c. Seismic Risk (Fault Proximity, Historical Hazard)

| Dimension | Specification |
|---|---|
| **Unit of Analysis** | Olympic venues (point), access corridors (100 m–1 km buffer) |
| **Time Window** | Historical seismic catalog (50+ years). Mapped faults (static). |
| **Cadence** | Not time-dependent; hazard is static unless earthquake occurs. |
| **Variable Meaning** | Distance to mapped fault (km). Historical earthquake frequency and magnitude. Ground-motion amplification (soil type, depth to bedrock). Peak ground acceleration (PGA) likely during event (probabilistic). |
| **Effect Size** | High-risk zone: PGA > 0.4 g during 2–3 year window (rare but possible). Moderate-risk: PGA 0.2–0.4 g (plausible). |
| **Latency** | Planning (months/years ahead). Earthquake forecasting is not possible at event timescale. |

**Would Work**
- USGS fault map (static, mapped traces)
- Earthquake historical catalogs (USGS, Southern California Seismic Network)
- Ground-motion prediction equations (empirical hazard models)
- Soil-type maps + rock depth from geology surveys
- ShakeMaps from past earthquakes (ground-motion validation)
- Maybe InSAR for recent creep on faults (active monitor)

**Wouldn't Work**
- Satellite thermal/optical for fault detection: faults not visible unless active (earthquakes, scarps)
- Real-time earthquake forecasting: not feasible scientifically
- Prediction of specific earthquake date/time: not possible

#### 10d. Wildfire Risk

| Dimension | Specification |
|---|---|
| **Unit of Analysis** | Urban/wildland interface (1–10 km from Olympic venues). Fire-prone zones (hillsides, chaparral). |
| **Time Window** | Fire season (May–November, worst June–Oct). Historical fire perimeters (30+ years). |
| **Cadence** | Seasonal (fire-weather forecast, 7–14 day outlook). Real-time during active fire (if occurs). |
| **Variable Meaning** | Fuel moisture (proxy: vegetation greenness NDVI, soil moisture, relative humidity). Fuel load (biomass). Fire danger index (FWI, combines temp, humidity, wind, drought). Burn scar location/extent (from previous fires). |
| **Effect Size** | Fire-weather danger ≥ "Extreme" on standard indices. Proximity to venues < 50 km (plume would degrade air quality). |
| **Latency** | Seasonal forecast 1–2 weeks. Real-time smoke tracking if active fire. |

**Would Work**
- Sentinel-2 / Landsat NDVI (fuel greenness proxy)
- MODIS fire detection (active burns, thermal anomalies)
- NFDIS fire perimeters (historical, recent)
- Fire-weather forecasts (NOAA, Cal Fire) + satellite fuel maps
- Smoke plume tracking (MODIS, Sentinel-5P, chem transport models)

**Wouldn't Work**
- Satellite alone (cannot predict fire ignition): need weather + fuel + spark
- Pre-season snapshot: must track seasonal dynamics

---

### Story 11: DC Metro Rail Expansion Under Heat Stress

**Persona Context**
- Transit operator, rail engineer, infrastructure manager
- Organization: WMATA (Washington Metropolitan Transit Authority)
- Expertise: rail operations, civil engineering (not remote sensing)
- Stakes: Service reliability, passenger safety, infrastructure lifespan

**Raw Problem**
"Monitor and predict thermal expansion of rail infrastructure under increasing heat. Plan interventions to prevent buckling and service failures."

**Need Statement**

| Dimension | Specification |
|---|---|
| **Unit of Analysis** | Specific rail segments or rail bed (10 m–10 km). Also broader track network (100+ km) for systemic risk assessment. |
| **Time Window** | Seasonal (summer peak heat). Long-term (decade-plus) for trend of increasing peak temperatures. |
| **Cadence** | Daily during heat season. |
| **Variable Meaning** | Rail temperature (°C, directly measured or inferred from air temp + solar load + rail color). Also ambient air temperature, solar intensity. Predict expansion (mm) as function of ΔT. |
| **Effect Size** | Rail temperature rise ≥20°C above baseline (induces ~240 mm expansion per km of rail with typical coefficients). Buckling risk at ≥ 50°C rail temp differential. |
| **Latency** | Forecast 3–7 days ahead for heat waves. Real-time monitoring (hourly). |

**Gotchas & Traps**
- **Scale trap:** Rail is ~10 cm wide; satellite pixels (10–100 m) cannot resolve individual rail. Must use ground-based sensors (thermometers along track) or drone thermal.
- **Material trap:** Rail temperature depends on color (dark rail absorbs more), shade, and time of day (afternoon peak ≠ 6 am). One-time satellite thermal snapshot insufficient; need diurnal cycle.
- **Expansion trap:** Thermal expansion is determined by rail temperature, not air temperature. Air temp can be 40°C, but rail in shade is 50°C and rail in full sun is 75°C. Large spatial heterogeneity.
- **Buckling trap:** Buckling risk is not just peak temp, but combination of temperature, rail alignment, track support stiffness, and existing internal stress. Cannot predict without detailed track model.
- **Trap for naive picker:** "Rail expansion" → temperature satellite → assumes homogeneous rail. In practice, rail yards have complex thermal environment (switches, joints, varying shade).

**Would Work**
- Ground-based fiber-optic temperature sensors along rail (distributed, direct measurement)
- Thermal camera (drone or fixed, 10 cm resolution) for localized hot spots
- High-resolution air temperature station network (hourly forecast)
- Detailed track model (FEM) forced with measured rail temperature
- WMATA's own instrumentation (strain gauges, displacement sensors)

**Wouldn't Work**
- Satellite thermal (10–100 m pixels): misses rail detail
- Single air-temperature forecast: does not account for rail-specific solar loading
- General heat-stress alerts: do not translate to specific track risk
- Historical statistics: trending temperature but not sufficient for real-time prediction

---

## Summary of Enrichment Patterns

Across all 11 stories, common themes emerge:

1. **Scale mismatch:** Most stories involve features (rivers, roads, buildings, structures) that are < 100 m, but satellite pixels are 10–1000 m. Either accept coarse data, use drone/airborne alternatives, or reframe the question to landscape scale.

2. **Temporal mismatch:** Many stories need real-time or sub-daily updates (SAR operations, volcanic plumes, rail heat). Satellite revisit (1–8 days) may not suffice; geostationary or ground-based monitoring required.

3. **Spectral/derivation requirements:** Nearly all stories need derived products (indices, decomposition, models), not raw satellite data. The algorithm and calibration (which trap-detection avoids) is half the solution.

4. **Ground truth as gating factor:** Stories with sparse in-situ networks (permafrost, deep ocean, remote areas) face large uncertainty from satellite alone. Local knowledge and field data essential.

5. **Latency matters:** Operational decisions (SAR, fire, heat stress) need < 6-hour latency; planning decisions tolerate weeks. Must state upfront or specification is incomplete.

6. **The "trap" is where satellites fail:** Each story reveals where naive application of satellite data fails (cloud, seasonal confounds, scale, spectral confusion). The librarian's job is to surface these **before** the user wastes time downloading.

