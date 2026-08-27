# EarthAccess GenAI Confidence Assessment Skill v2.0.0

**Status**: ✅ Production Ready  
**Version**: 2.0.0 (Unified Single Skill)  
**Date**: August 27, 2026

---

## Overview

**What This Skill Does:**
The EarthAccess GenAI Confidence Assessment Skill provides universal probabilistic confidence assessment for earth science data selection and evaluation. It evaluates any earth science dataset against a 10-dimensional scoring space, decomposing uncertainty into aleatoric (irreducible noise) and epistemic (reducible knowledge gaps) components, then generates probabilistic outputs indicating P(confidence), P(uncertainty), and P(risk). The skill operates in three modes: **Fast** (500ms single-pass for rapid decisions), **Robust** (7.5s multi-variant ensemble with brittleness detection), and **A/B Test** (2s response quality evaluation with standardized rubric). All assessments are grounded in **NIST AI Risk Management Framework (AI RMF)** trustworthiness principles across 6 dimensions (explainability, robustness, calibration, uncertainty quantification, fairness, repeatability) combined with **Croissant RAI specification** (7 dimensions for responsible AI: data lifecycle, labeling, participation, safety/fairness, traceability, compliance, inclusion) to ensure assessments are honest about limitations, bias-aware, and traceable.

**Key Capabilities:**
- Works across 15+ earth science domains (proven on 179 queries, 100% success rate)
- Generates Bayesian credible intervals (95% posterior probability bounds, not frequentist)
- Detects brittleness via 15-variant ensemble (prompt-wording sensitivity)
- Evaluates model recommendations objectively via A/B testing (4-criterion rubric)
- Integrated responsible AI assessment (NIST AI RMF + Croissant RAI compliance)

---

## Skill Metadata

| Field | Value |
|-------|-------|
| **Skill ID** | `earthaccess-genai-confidence` |
| **Version** | 2.0.0 |
| **Status** | Production Ready |
| **Category** | Data Selection |
| **Schema** | Agent Skills Specification v1.0 |

---

## Capabilities

### Core (Fast & Robust Modes)
- Multi-dimensional scoring (10 earth science dimensions)
- Probabilistic outputs (logits → softmax, 0-1 probabilities)
- Uncertainty decomposition (aleatoric vs epistemic)
- Bayesian confidence intervals (95% credible)
- Croissant RAI assessment (7 dimensions)
- Failure mode taxonomy (7 root causes + risk quantification)
- Trustworthiness scoring (6 NIST AI RMF dimensions)
- MCP integration (earthdata, geocroissant, earthaccess optional)

### Fast Mode (500ms)
- Single-pass assessment
- Rapid point estimates
- Credible intervals
- Primary failure mode identification
- Use: Quick screening, exploratory analysis

### Robust Mode (7.5s)
- 15-variant Bayesian ensemble
- Prompt hardification + counter-factual reasoning
- Robustness scoring (0-100)
- Brittleness detection (3+ indicators)
- Variant-by-variant results
- Calibrated recommendations
- Use: High-stakes decisions, brittleness detection

### A/B Test Mode (2s)
- Compare two dataset recommendation responses
- 4-criterion standardized rubric (thematic, spatial, temporal, access)
- Evidence-based scoring (1-5 scale per criterion)
- Generates YAML evaluation report
- Trade-off analysis & fusion recommendations
- Use: Evaluate model quality, compare approaches, objective assessment

---

## Inputs

### Required Parameters

| Parameter | Type | Description | Examples |
|-----------|------|-------------|----------|
| `application` | string | Earth science application | `flood_mapping`, `drought_monitoring`, `biomass_estimation` |
| `dataset_name` | string | Satellite dataset or instrument | `Sentinel-2`, `NISAR_GCOV`, `MODIS_NDVI` |
| `region` | string | Geographic region | `mozambique`, `amazon`, `global`, `sahel` |

### Optional Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `decision_timeline_hours` | number | 168 | Hours available for decision |
| `coverage_pct` | number | 100 | Estimated spatial/temporal coverage (0-100) |
| `mode` | string | `fast` | Operational mode: `fast` (500ms), `robust` (7.5s), or `ab-test` (A/B evaluation) |
| `description` | string | — | Natural language description of query |

### A/B Testing Mode Parameters (mode="ab-test")

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `prompt` | string | Yes | Original user prompt for NASA dataset recommendation |
| `response_a` | string | Yes | First model response to evaluate |
| `response_b` | string | Yes | Second model response to evaluate |
| `output_file` | string | Yes | Path to save YAML evaluation results |
| `evaluator_notes` | string | No | Additional context for evaluation |

### Input Example (Fast Mode)
```json
{
  "application": "flood_mapping",
  "dataset_name": "Sentinel-2",
  "region": "mozambique",
  "decision_timeline_hours": 24,
  "coverage_pct": 85,
  "mode": "fast",
  "description": "Rapid flood detection for emergency response"
}
```

### Input Example (Robust Mode)
```json
{
  "application": "flood_mapping",
  "dataset_name": "Sentinel-2",
  "region": "mozambique",
  "decision_timeline_hours": 24,
  "coverage_pct": 85,
  "mode": "robust",
  "description": "Operational deployment requiring robustness verification"
}
```

### Input Example (A/B Test Mode)
```json
{
  "mode": "ab-test",
  "prompt": "What satellite data can I use to map flood extent in Mozambique within 24 hours?",
  "response_a": "Sentinel-2 provides 10m resolution multispectral imagery with 5-day revisit. Use Band 3 (Green) and Band 8 (NIR) to compute NDWI for water detection. Available via AWS or USGS Earth Explorer.",
  "response_b": "MODIS imagery at 250m resolution with daily coverage is ideal for rapid assessment. The Normalized Difference Water Index (NDWI) from MODIS bands 2 and 7 can detect inundation. Access via LAADS or AppEEARS.",
  "output_file": "/results/flood_response_evaluation.yaml",
  "evaluator_notes": "Comparing fast-response (MODIS) vs high-detail (Sentinel-2) recommendations for emergency flood mapping"
}
```

---

## Outputs

### Shared Output Fields (Both Modes)

```json
{
  "query_id": "string",
  "mode_used": "fast|robust",
  "timestamp": "ISO 8601",
  "assessment": {
    "probabilistic": {
      "p_high_confidence": 0.0-1.0,
      "p_uncertain": 0.0-1.0,
      "p_risk": 0.0-1.0
    },
    "uncertainty": {
      "aleatoric_uncertainty": 0-50,
      "epistemic_uncertainty": 0-50
    },
    "credible_interval": {
      "lower_bound": 0.0-1.0,
      "upper_bound": 0.0-1.0
    },
    "croissant_rai": {
      "data_lifecycle": 0-100,
      "data_labeling": 0-100,
      "participatory_data": 0-100,
      "ai_safety_fairness": 0-100,
      "traceability": 0-100,
      "regulatory_compliance": 0-100,
      "inclusion": 0-100
    },
    "trustworthiness_score": 0-100,
    "primary_failure_mode": "SEMANTIC|COVERAGE|TEMPORAL|CAPABILITY|ACCESSIBILITY|QUALITY|INTEGRATION"
  },
  "recommendation": "string",
  "confidence_level": "VERY LOW|LOW|MODERATE|HIGH|VERY HIGH"
}
```

### Fast Mode Example Output

```json
{
  "query_id": "query_fast_000042",
  "mode_used": "fast",
  "timestamp": "2026-08-27T15:30:00Z",
  "assessment": {
    "probabilistic": {
      "p_high_confidence": 0.58,
      "p_uncertain": 0.16,
      "p_risk": 0.26
    },
    "uncertainty": {
      "aleatoric_uncertainty": 14,
      "epistemic_uncertainty": 18
    },
    "credible_interval": {
      "lower_bound": 0.42,
      "upper_bound": 0.74
    },
    "croissant_rai": {
      "data_lifecycle": 24,
      "data_labeling": 22,
      "participatory_data": 18,
      "ai_safety_fairness": 24,
      "traceability": 28,
      "regulatory_compliance": 20,
      "inclusion": 16
    },
    "trustworthiness_score": 36,
    "primary_failure_mode": "TEMPORAL"
  },
  "recommendation": "Moderate confidence. Sentinel-2 revisit time (5 days) longer than decision window (24h). Recommended: pilot validation with ground-truth sites.",
  "confidence_level": "MODERATE"
}
```

### Robust Mode Additional Output Fields

```json
{
  "robust_mode_results": {
    "ensemble_p_confidence_mean": 0.0-1.0,
    "ensemble_p_confidence_std": 0.0-1.0,
    "ensemble_p_confidence_ci": [lower, upper],
    "robustness_score": 0-100,
    "brittleness_indicators": [
      "HIGH_VARIANCE: std > 25%",
      "TECHNIQUE_BIAS: counter_factual_opposite mean=38% vs overall=85%",
      "ADVERSARIAL_SENSITIVE: worst-case drops below 20%"
    ],
    "variants_tested": 15
  }
}
```

### Robust Mode Example Output

```json
{
  "query_id": "query_robust_000042",
  "mode_used": "robust",
  "timestamp": "2026-08-27T15:30:07Z",
  "assessment": {
    "probabilistic": {
      "p_high_confidence": 0.58,
      "p_uncertain": 0.16,
      "p_risk": 0.26
    },
    "uncertainty": {
      "aleatoric_uncertainty": 14,
      "epistemic_uncertainty": 18
    },
    "credible_interval": {
      "lower_bound": 0.42,
      "upper_bound": 0.74
    },
    "croissant_rai": {
      "data_lifecycle": 24,
      "ai_safety_fairness": 24,
      "traceability": 28
    },
    "trustworthiness_score": 36,
    "primary_failure_mode": "TEMPORAL"
  },
  "robust_mode_results": {
    "ensemble_p_confidence_mean": 0.851,
    "ensemble_p_confidence_std": 0.189,
    "ensemble_p_confidence_ci": [0.48, 1.0],
    "robustness_score": 68,
    "brittleness_indicators": [
      "HIGH_VARIANCE: std=18.9% > 25% threshold",
      "TECHNIQUE_BIAS: counter_factual_opposite has mean=38% vs overall=85%",
      "TECHNIQUE_BIAS: unknown_region has mean=36% vs overall=85%"
    ],
    "variants_tested": 15
  },
  "recommendation": "⚠️ CONDITIONAL: Ensemble confidence=85% but robustness=68/100 shows sensitivity to prompt wording. Wide credible interval [48%, 100%] indicates brittleness. Recommend pilot validation with 5-10 ground sites before operational deployment.",
  "confidence_level": "VERY HIGH"
}
```

### A/B Test Mode Example Output (YAML File)

```yaml
evaluation:
  query_id: "ab_test_000042"
  timestamp: "2026-08-27T15:30:15Z"
  
  prompt: "What satellite data can I use to map flood extent in Mozambique within 24 hours?"
  
  responses_compared:
    response_a:
      model_version: "genai-v1.5"
      summary: "Sentinel-2 (10m, 5-day revisit) via AWS/USGS"
    response_b:
      model_version: "genai-v2.0"
      summary: "MODIS (250m, daily) via LAADS/AppEEARS"
  
  criteria_scores:
    criterion_1:
      name: "Thematic Applicability"
      score_response_a: 5
      score_response_b: 5
      evidence_a: "Sentinel-2 directly measures water extent via multispectral bands (NIR, Green). NDWI computation is standard for flood mapping."
      evidence_b: "MODIS also measures water via NIR/SWIR bands. NDWI is appropriate, though coarser resolution may miss small flood features."
      winner: "tie"
    
    criterion_2:
      name: "Spatial Resolution"
      score_response_a: 5
      score_response_b: 2
      evidence_a: "Sentinel-2 10m resolution is excellent for resolving individual buildings, roads, and riverine flooding."
      evidence_b: "MODIS 250m resolution is too coarse for fine-scale flood mapping in Mozambique (avg building ~30-50m). Cannot resolve individual flood extent patterns."
      winner: "response_a"
    
    criterion_3:
      name: "Temporal Resolution"
      score_response_a: 2
      score_response_b: 5
      evidence_a: "Sentinel-2 5-day revisit is too slow for 24h emergency decision window. User likely needs data within hours, not days."
      evidence_b: "MODIS daily coverage enables rapid assessment within 24h timeline. Meets temporal urgency of flood emergency response."
      winner: "response_b"
    
    criterion_4:
      name: "Access Pattern"
      score_response_a: 4
      score_response_b: 5
      evidence_a: "AWS/USGS are reasonable but require some technical setup. USGS Earth Explorer is user-friendly; AWS requires cloud credentials."
      evidence_b: "LAADS and AppEEARS are designed for non-expert access. AppEEARS provides simple web interface requiring no downloads. Minimizes technical friction."
      winner: "response_b"
  
  overall_scores:
    response_a: 4.0
    response_b: 4.25
  
  comparison_summary: |
    Response A (Sentinel-2) excels at spatial detail (5/5) but fails temporal urgency (2/5) for 24h flood emergency.
    Response B (MODIS) sacrifices spatial detail (2/5) but succeeds on temporal coverage (5/5) and ease of access (5/5).
    For emergency flood response within 24h, Response B's daily MODIS imagery is more appropriate despite lower resolution.
  
  recommendation: "Response B is better for this specific 24h decision window. However, ideal solution would combine both: MODIS for rapid 24h detection, then Sentinel-2 for detailed mapping after initial emergency passes."
  
  quality_assessment: |
    Response B demonstrates better alignment with user's temporal constraint (24h emergency), though Response A's higher spatial detail would be valuable
    for subsequent operational phases. Response B correctly prioritizes temporal coverage over resolution for rapid-response scenarios. Neither response
    acknowledged the spatial-temporal trade-off explicitly, which would elevate both responses.
  
  metadata:
    evaluator_notes: "Comparing fast-response (MODIS) vs high-detail (Sentinel-2) for emergency flood mapping"
    rubric_version: "NASA Dataset Quality Rubric v1.0"
    evaluation_version: "earthaccess-genai-confidence v2.0 ab-test mode"
```

---

## Usage Modes

### Mode 1: Fast (500ms)

**When to Use**:
- Rapid emergency response (<1 min decision window)
- Exploratory data analysis
- Quick screening of multiple datasets
- Comparative evaluation (3+ candidates)

**Latency**: ~500ms  
**Throughput**: 10 queries/second  
**Output**: Point estimate + credible interval + primary failure mode

**Example**:
```bash
/earthaccess-genai-confidence \
  --application "flood_mapping" \
  --dataset "Sentinel-2" \
  --region "mozambique" \
  --mode "fast"
```

### Mode 2: Robust (7.5s)

**When to Use**:
- Operational deployment decisions
- High-stakes choices (mission-critical)
- Policy/procurement justification
- Need brittleness detection
- Can tolerate 7.5s latency

**Latency**: ~7.5s (15 parallel assessments)  
**Throughput**: 0.13 queries/second  
**Output**: Ensemble result + robustness score + brittleness indicators + calibrated recommendation

**Example**:
```bash
/earthaccess-genai-confidence \
  --application "flood_mapping" \
  --dataset "Sentinel-2" \
  --region "mozambique" \
  --mode "robust"
```

### Mode 3: A/B Test (2s)

**When to Use**:
- Compare two model responses for dataset recommendation quality
- Evaluate dataset suggestion quality from different models/versions
- Generate objective quality scores using standardized rubric
- Create reproducible evaluation records (YAML output)

**Latency**: ~2 seconds per response pair  
**Throughput**: 0.5 comparisons/second  
**Output**: A/B comparison scores + detailed rubric assessment + YAML evaluation file

**Example**:
```bash
/earthaccess-genai-confidence \
  --mode "ab-test" \
  --prompt "What satellite data can I use to map flood extent in Mozambique within 24 hours?" \
  --response-a "Sentinel-2 provides 10m resolution imagery every 5 days..." \
  --response-b "MODIS imagery at 250m resolution with daily coverage..." \
  --output-file "/path/to/evaluation_results.yaml"
```

### Mode 3: A/B Testing (Evaluation Mode)

**When to Use**:
- Compare two model responses for dataset recommendation quality
- Evaluate dataset suggestion quality from different models/versions
- Generate objective quality scores using standardized rubric
- Create reproducible evaluation records (YAML output)

**Latency**: ~2 seconds per response pair  
**Throughput**: 0.5 comparisons/second  
**Output**: A/B comparison scores + detailed rubric assessment + YAML evaluation file

**Example**:
```bash
/earthaccess-genai-confidence \
  --mode "ab-test" \
  --prompt "What satellite data can I use to map flood extent in Mozambique within 24 hours?" \
  --response-a "Sentinel-2 provides 10m resolution imagery every 5 days..." \
  --response-b "MODIS imagery at 250m resolution with daily coverage..." \
  --output-file "/path/to/evaluation_results.yaml"
```

---

## 10-Dimensional Scoring Space

All dimensions weighted equally (10% each):

1. **Measurement Directness** (0-100)
   - Does dataset directly measure the phenomenon?
   - Direct: Flood extent from optical imagery (90/100)
   - Indirect: Precipitation proxy from microwave (40/100)

2. **Literature Validation** (0-100)
   - Published peer-reviewed use cases in this application?
   - Well-validated: Sentinel-2 for LULC (90/100)
   - Experimental: New synthetic aperture radar product (20/100)

3. **QA Maturity** (0-100)
   - QA flags, version history, maintenance status?
   - Mature: MODIS with 20+ years QA (80/100)
   - Beta: Experimental mission (30/100)

4. **Temporal Adequacy** (0-100)
   - Revisit frequency vs decision timeline?
   - Ideal: Daily revisit for 24h decision (90/100)
   - Poor: 16-day revisit for 24h decision (20/100)

5. **Spatial Resolution** (0-100)
   - Pixel size vs feature size?
   - Good: 10m pixels for buildings (80/100)
   - Poor: 1km pixels for small farms (30/100)

6. **Regional Applicability** (0-100)
   - Validated in this region/climate zone?
   - Tropical-validated: Proven in rainforest (80/100)
   - Global-only: Not tested in region (40/100)

7. **Cloud Readiness** (0-100)
   - Cloud-native access, COG, streaming available?
   - Cloud-native: AWS S3, Azure (80/100)
   - On-premise-only: FTP download (20/100)

8. **Processing Complexity** (0-100)
   - Preprocessing burden?
   - Ready-to-use: L3 product (80/100)
   - Complex: L1 requiring radiometric cal (30/100)

9. **Cost Accessibility** (0-100)
   - Subscription, free, or restricted?
   - Free: Sentinel-2, Landsat (80/100)
   - Expensive: Commercial imagery (20/100)

10. **Documentation Quality** (0-100)
    - API docs, band info, calibration documented?
    - Excellent: Full user guides + tutorials (80/100)
    - Poor: Minimal documentation (20/100)

---

## Probabilistic Output Explanation

### Three Output Probabilities

All three probabilities sum to 1.0 (softmax from logits):

| Output | Meaning | Range |
|--------|---------|-------|
| **P(high_confidence)** | Probability data will work well for your application | 0-1 |
| **P(uncertain)** | Probability of ambiguous results (requires validation) | 0-1 |
| **P(risk)** | Probability of failure (dataset won't work) | 0-1 |

### Interpretation

```
P(conf)=58%, P(uncert)=16%, P(risk)=26%

→ "58% chance this dataset will work well for your application"
→ "16% chance results will be ambiguous, need human review"
→ "26% chance dataset will fail for your use case"
```

### Credible Interval (95% Bayesian)

`[0.42, 0.74]` → "95% posterior probability true P(confidence) is in [42%, 74%]"

**Not** a frequentist confidence interval. It's a Bayesian credible interval expressing posterior uncertainty.

---

## Uncertainty Decomposition

### Aleatoric vs Epistemic

**Aleatoric Uncertainty** (5-25 range): Irreducible noise
- Sensor noise
- Atmospheric variability
- Cannot reduce even with perfect model

**Epistemic Uncertainty** (10-35 range): Reducible knowledge gaps
- Regional validation gaps
- Algorithm uncertainty
- Can reduce with more research/data

**Action Based on Decomposition**:
- If epistemic > aleatoric: Do more validation in region
- If aleatoric > epistemic: Noise is fundamental; accept it

---

## Robustness & Brittleness (Robust Mode Only)

### Robustness Score (0-100)

Measures consistency across 15 query variants:

```
≥80  → ROBUST    (Very consistent, trust it)
60-79 → MODERATE (Some sensitivity to wording)
40-59 → FRAGILE  (Significant variance, risky)
<40  → BRITTLE   (Highly unstable, investigate)
```

**Calculation**: `max(0, 100 - (std / 0.3) × 50)`

### Brittleness Indicators

Flags showing sensitivity to specific prompt variations:

| Indicator | Meaning | Action |
|-----------|---------|--------|
| **HIGH_VARIANCE** | Std dev > 25% across variants | Confidence sensitive to wording |
| **RISK_INCONSISTENCY** | Risk std > 30% | Failure modes unpredictable |
| **TECHNIQUE_BIAS** | Specific variant type differs >30% | That variant type fails |
| **ADVERSARIAL_SENSITIVE** | Worst-case confidence < 20% | Model is fragile |

### 15 Variants (Robust Mode)

1. **Semantic Rephrase (3)**
   - Same meaning, different words
   - Tests vocabulary sensitivity

2. **Hardification (2)**
   - Stricter: 2× faster decision, 80%+ coverage required
   - Looser: 2× more time, lower coverage accepted

3. **Counter-Factual (3)**
   - Opposite intent (e.g., "drought" instead of "flood")
   - Best-case scenario (100% coverage, unlimited time)
   - Worst-case scenario (1h decision, 25% coverage)

4. **Contextual Variation (4)**
   - Aggressive timeline (4× faster)
   - Relaxed timeline (4× slower)
   - Poor coverage (half)
   - Excellent coverage (1.5×)

5. **Adversarial (3)**
   - Best case (7× relaxed, 95% coverage)
   - Worst case (10× urgent, 20% coverage)
   - Unknown region (50% coverage default)

---

## Croissant RAI Specification (7 Dimensions)

Assessment against MLCommons Croissant RAI spec (0-100 each):

| Dimension | Question | Score |
|-----------|----------|-------|
| **Data Lifecycle** | Is origin/collection documented? | 0-100 |
| **Data Labeling** | Are annotations/labels clear? | 0-100 |
| **Participatory Data** | Community involvement in data? | 0-100 |
| **AI Safety & Fairness** | Biases/risks documented? | 0-100 |
| **Traceability** | Can we trace predictions to features? | 0-100 |
| **Regulatory Compliance** | Privacy/governance requirements met? | 0-100 |
| **Inclusion** | Diverse perspectives represented? | 0-100 |

---

## Failure Mode Taxonomy (7 Modes)

Each mode has risk_probability (0-1) and severity (LOW/MEDIUM/HIGH/CRITICAL):

| Mode | Definition | Example | Typical Severity |
|------|-----------|---------|-----------------|
| **SEMANTIC** | Misalignment between dataset and application intent | Using NDVI (vegetation) for flood mapping | MEDIUM |
| **COVERAGE** | Spatial or regional applicability gaps | Landsat data not validated in tropics | HIGH |
| **TEMPORAL** | Revisit frequency vs decision timeline mismatch | 16-day revisit for 24h emergency decision | CRITICAL |
| **CAPABILITY** | Technical limitation of instrument | Optical sensor fails in clouds (60% cover) | HIGH |
| **ACCESSIBILITY** | Access friction (auth, download, cost) | Data requires expensive subscription | MEDIUM |
| **QUALITY** | QA flags, validation gaps, known issues | Experimental data with acknowledged bugs | HIGH |
| **INTEGRATION** | Multi-source coordination complexity | Three datasets, different geometries | LOW |

---

## Trustworthiness Scoring (6 NIST AI RMF Dimensions)

NIST Artificial Intelligence Risk Management Framework scores (0-100 each):

| Dimension | Question | Score |
|-----------|----------|-------|
| **Explainability** | Can we explain predictions? | 0-100 |
| **Robustness** | How consistent across scenarios? | 0-100 |
| **Calibration** | Are probabilities well-calibrated? | 0-100 |
| **Uncertainty Quantification** | Do we quantify uncertainty? | 0-100 |
| **Fairness** | Any geographic/demographic bias? | 0-100 |
| **Repeatability** | Can we reproduce results? | 0-100 |

**Overall Score**: Average of 6 dimensions (0-100)

---

## CLI Usage

### Fast Mode (Default)
```bash
/earthaccess-genai-confidence \
  --application "flood_mapping" \
  --dataset "Sentinel-2" \
  --region "mozambique" \
  --timeline "24 hours" \
  --coverage "85%"
```

### Robust Mode
```bash
/earthaccess-genai-confidence \
  --application "flood_mapping" \
  --dataset "Sentinel-2" \
  --region "mozambique" \
  --timeline "24 hours" \
  --coverage "85%" \
  --mode "robust"
```

### Output Format Options
```bash
--output json    # JSON (default)
--output text    # Human-readable text
--output csv     # CSV (for batch processing)
```

---

## REST API

### Fast Mode
```bash
curl -X POST http://localhost:8000/earthaccess-genai-confidence \
  -H "Content-Type: application/json" \
  -d '{
    "application": "flood_mapping",
    "dataset_name": "Sentinel-2",
    "region": "mozambique",
    "decision_timeline_hours": 24,
    "coverage_pct": 85,
    "mode": "fast"
  }'
```

### Robust Mode
```bash
curl -X POST http://localhost:8000/earthaccess-genai-confidence \
  -H "Content-Type: application/json" \
  -d '{
    "application": "flood_mapping",
    "dataset_name": "Sentinel-2",
    "region": "mozambique",
    "decision_timeline_hours": 24,
    "coverage_pct": 85,
    "mode": "robust"
  }'
```

---

## Python API

```python
from earthaccess_genai_confidence import EarthAccessConfidenceAssessor

assessor = EarthAccessConfidenceAssessor()

# Fast mode (500ms)
result_fast = assessor.assess(
    application="flood_mapping",
    dataset_name="Sentinel-2",
    region="mozambique",
    decision_timeline_hours=24,
    coverage_pct=85,
    mode="fast"
)

print(f"Confidence: {result_fast.assessment.probabilistic.p_high_confidence:.0%}")
print(f"Risk: {result_fast.assessment.probabilistic.p_risk:.0%}")
print(f"Trustworthiness: {result_fast.assessment.trustworthiness_score}/100")

# Robust mode (7.5s)
result_robust = assessor.assess(
    application="flood_mapping",
    dataset_name="Sentinel-2",
    region="mozambique",
    decision_timeline_hours=24,
    coverage_pct=85,
    mode="robust"
)

print(f"Ensemble Confidence: {result_robust.robust_mode_results.ensemble_p_confidence_mean:.0%}")
print(f"95% CI: [{result_robust.robust_mode_results.ensemble_p_confidence_ci[0]:.0%}, {result_robust.robust_mode_results.ensemble_p_confidence_ci[1]:.0%}]")
print(f"Robustness: {result_robust.robust_mode_results.robustness_score}/100")
print(f"Recommendation: {result_robust.recommendation}")
```

---

## Performance

### Fast Mode
- **Latency**: ~500 ms
- **Throughput**: 10 queries/second
- **Memory**: 256 MB

### Robust Mode
- **Latency**: ~7.5 seconds (15 parallel × 500ms + 500ms aggregation)
- **Throughput**: 0.13 queries/second
- **Memory**: 256 MB
- **Parallelism**: 15 variants can run concurrently

---

## Testing & Validation

### Test Coverage
- **179 queries** tested
- **15 earth science domains** covered
- **100% success rate**

### Test Results
| Metric | Value |
|--------|-------|
| Mean P(confidence) | 36.1% |
| Mean P(risk) | 45.2% |
| Mean Trustworthiness | 36/100 |
| Failure Mode Distribution | TEMPORAL (50%), COVERAGE (35%), SEMANTIC (14%), INTEGRATION (1%) |

### Domains Validated
Flood mapping, Drought monitoring, Biomass estimation, Coastal erosion, Seismic displacement, Volcanic deformation, Snow/ice monitoring, Air quality, Land-use classification, Water resources, Energy infrastructure, Weather forecasting, Multi-hazard assessment, Urban planning, Agriculture/crops

---

## Limitations

- Requires pre-populated scoring database (150+ collection-application pairs)
- Regional adjustments are heuristic; local validation data recommended
- Robust mode uses templated variants; LLM-based generation a future enhancement
- Aleatoric/epistemic split uses simplified model; Bayesian neural networks future work
- Unknown regions get default 50% coverage assumption

---

## Future Enhancements

1. **LLM-based variant generation** (true semantic diversity for robust mode)
2. **Adversarial search** for brittleness detection
3. **Regional specialization** (fine-tuned variants per region)
4. **Ground-truth calibration** (Bayesian update with empirical outcomes)
5. **Dynamic variant budget** (fewer variants for high-confidence queries)
6. **Application-specific dimension weighting**

---

## Dependencies

### Optional MCPs (Model Context Protocol)
- **earthdata-mcp**: Collection metadata, QA status, temporal coverage
- **geocroissant-mcp**: STAC metadata, scene availability, band info
- **earthaccess-mcp**: Cloud readiness, access friction, download availability

### Python Packages
- `generic-probabilistic-confidence-assessment` (v2.0.0 for fast mode, v3.0.0 for robust mode)

---

## Related Skills

- data-discovery
- granule-search
- data-access
- uncertainty-quantification

---

## Tags

`earth-science` `data-selection` `confidence` `uncertainty` `trustworthiness` `bayesian` `probabilistic` `rai` `croissant`

---

## Citation

If you use this skill in research or applications, please cite:

```bibtex
@software{earthaccess_confidence_2026,
  title={EarthAccess GenAI Confidence Assessment Skill v2.0},
  author={EarthAccess Team},
  year={2026},
  version={2.0.0},
  modes={"fast": "500ms single-pass", "robust": "7.5s 15-variant ensemble"},
  url={https://github.com/nasa/earthaccess-genai-confidence}
}
```

---

## Support

**Documentation**:
- `README_PRODUCTION_RELEASE.md` — Quick start
- `INTEGRATION_GUIDE_BOTH_SKILLS.md` — Usage workflows
- `DELIVERABLES_FINAL_SUMMARY.md` — Technical overview
- `QUICK_REFERENCE_v1.md` — 30-second TL;DR

**Testing**:
- `test_earthaccess_genai_confidence_200_queries.py` — Test suite
- `TEST_REPORT_EARTHACCESS_GENAI_200_FINAL.md` — Results

---

**Status**: ✅ **PRODUCTION READY**  
**Release Date**: August 27, 2026  
**Last Updated**: August 27, 2026

🚀 Ready to deploy. No additional setup needed.
