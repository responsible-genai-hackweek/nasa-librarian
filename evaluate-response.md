# Evaluate NASA Dataset Response Quality

## Purpose
Evaluate the quality of a model's suggested datasets to answer a particular scientific question. Frame your evaluation around the thematic, spatial resolution, temporal resolution, and data access criteria given below. Score each criterion independently, combine these scores into an overall score, and document your findings in a YAML output file.

## Inputs Required
- **Prompt**: The original prompt that was given to the model
- **Response**: The model's response to be evaluated
- **Output File**: Path where the evaluation results should be saved as YAML

## Evaluation Process

### Step 1: Review Prompt and Response
The prompt emulates a user that is trying to identify a dataset for their purpose from NASA Earthdata. Your evaluation should determine whether the response gave reasonable suggestions to the user.

Carefully read and understand:
1. What the prompt is asking for. Be sure to consider the appropriate spatial resolution and temporal resolution for their question. Fine-scale phenomena (like rivers and individual buildings) cannot be investigated with data at a coarse spatial resolution.
2. What the model actually produced. Determine what datasets that model suggested and what applications is suggested them for. If multiple datasets were suggested, your evaluation should weight the earliest responses as more important than the later ones.

### Step 2: Evaluate Each Criterion
For each criterion in the rubric below:
1. Read the criterion description and scoring guidance
2. Examine the response against that criterion
3. Assign a score on a Likert scale (1-5, with 5 being high-quality).
4. Document specific observations and evidence supporting the score

### Step 3: Calculate Overall Score
Combine the criterion scores using equal weighting:
- **Overall Score** = (Sum of all criterion scores) / (Number of criteria)
- Round to one decimal place

### Step 4: Document Commentary
Write a brief summary (2-3 sentences) of:
- The response's primary strengths
- Any significant weaknesses or areas for improvement
- Overall assessment of quality

### Step 5: Output Results
Save results to the specified YAML file in the format shown below.

## Rubric Template

### Criterion 1: Thematic applicability
- **Scale**: 1-5
- **Description**: Do the suggested datasets measure the phenomenon or process the user is interested in?
- **Scoring Guidance**:
  - 1: There is no feasible way that the user could learn anything from the suggested data.
  - 3: The suggested data is tangentially related to the user's interest, but requires careful interpretation.
  - 5: The suggested data is related to the user's interest and is the appropriate measurement for their application.

### Criterion 2: Spatial resolution
- **Scale**: 1-5
- **Description**: Are the suggested datasets on the appropriate spatial scale for the user's application?
- **Scoring Guidance**:
  - 1: There is a substantial mismatch between data resolution and the application (e.g., coarse climate model output for a neighborhood-scale analysis) that would harm the user's ability to learn anything from the data.
  - 3: The data resolution is similar to the application resolution, but may not be totally comparable.
  - 5: The data resolution is finer or of a similar scale to the application resolution and can adequately resolve the phenomena of interest.
- **NOTE**: For some applications, data at the appropriate spatial scale may not be available. If the response indicates this limitation, this should be considered a high-quality response.

### Criterion 3: Temporal resolution
- **Scale**: 1-5
- **Description**: Are the suggested datasets on the appropriate temporal scale for the user's application?
- **Scoring Guidance**:
  - 1: There is a substantial mismatch between data resolution and the application (e.g., annual imagery for a rapid-response analysis) that would harm the user's ability to learn anything from the data.
  - 3: The data resolution is similar to the application resolution, but may not be totally comparable.
  - 5: The data resolution is finer or of a similar scale to the application resolution and can adequately resolve the phenomena of interest.
- **NOTE**: For some applications, data at the appropriate temporal scale may not be available. If the response indicates this limitation, this should be considered a high-quality response.

### Criterion 4: Access pattern
- **Scale**: 1-5
- **Description**: Does the response indicate how the user can access the data of interest?
- **Scoring Guidance**:
  - 1: The access pattern in the response assumes greater technical proficiency than the user indicated, or the response suggests an access pattern outside of a NASA-maintained service.
  - 3: The suggested access pattern is reasonble for the user's technical proficiency, but may imply bottlenecks that would harm their analysis (e.g., downloading many satellite granules for viewing in QGIS is undesirable for time series analysis).
  - 5: The suggested access pattern is reasonable for the user's technical proficiency and minimizes the level of effort needed to view data for the application.

## Output YAML Format

```yaml
evaluation:
  prompt: |
    [Original prompt text]
  
  criteria_scores:
    criterion_1:
      name: "[Criterion Name]"
      score: [1-5]
      evidence: "[Specific observations supporting this score]"
    
    criterion_2:
      name: "[Criterion Name]"
      score: [1-5]
      evidence: "[Specific observations supporting this score]"
    
    criterion_3:
      name: "[Criterion Name]"
      score: [1-5]
      evidence: "[Specific observations supporting this score]"
  
  overall_score: [Calculated average, 1 decimal place]
  
  commentary: |
    [2-3 sentence summary of strengths, weaknesses, and overall quality assessment]
  
  timestamp: "[ISO 8601 date-time]"
```

## Notes
- Be objective and evidence-based in scoring
- If a criterion doesn't apply, document why and skip it from overall calculation
- Provide specific examples from the response to support each score
- Ensure the YAML is valid and properly formatted
