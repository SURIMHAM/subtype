# Residual-Based Cognitive Subgroup and SILA-Based Stage Definition Summary

## 1. Residual-Based TopResidual/Gap Subgroup Definition

### 1.1 Analytical Goal

The goal of the subgroup definition is to classify individuals by **relative cognitive profile**, not by global cognitive severity. This distinction is important because MEM, LAN, and EXF scores are all affected by disease severity. If raw scores or impairment z-scores are clustered directly, the resulting groups may mainly reflect mild versus severe impairment rather than domain-specific cognitive phenotype.

The current subgroup method therefore separates two concepts:

```text
Global cognitive burden: average impairment across MEM/LAN/EXF
Relative cognitive profile: which domain is worse than the individual's own average impairment
```

This is conceptually aligned with the subtype-stage separation logic discussed for SuStaIn-like analyses: subgroup should capture **phenotype/profile**, whereas stage should capture **progression/severity/time**.

### 1.2 Reference-Based Impairment Z-Score

The first step is to define domain-wise impairment scores using first-visit CN/AP-negative subjects as the reference group.

```matlab
Zimp = -(raw cognition - reference mean) ./ reference SD;
```

Because MEM, LAN, and EXF are cognitive scores where higher values indicate better performance, the sign is reversed. Therefore:

```text
Higher Zimp = greater cognitive impairment
Zimp = 0    = reference mean level
Zimp = 1    = 1 SD worse than reference
```

The domains used are:

```text
Zimp_MEM
Zimp_LAN
Zimp_EXF
```

The reference group used in the current implementation is:

```text
First-visit CN/AP-negative subjects, n = 262
```

### 1.3 Average Impairment and Residual Profile

For each observation, global cognitive burden is defined as the mean of the three impairment z-scores:

```matlab
AvgZ = mean([Zimp_MEM, Zimp_LAN, Zimp_EXF], 2);
```

Then the relative domain profile is calculated by subtracting the person's own average impairment:

```matlab
Resid_MEM = Zimp_MEM - AvgZ;
Resid_LAN = Zimp_LAN - AvgZ;
Resid_EXF = Zimp_EXF - AvgZ;
```

Interpretation:

```text
Resid_domain > 0 : that domain is worse than the person's average cognitive impairment
Resid_domain < 0 : that domain is relatively spared
Resid_domain ≈ 0 : that domain is close to the person's overall cognitive level
```

Because the residuals are centered within each person:

```text
Resid_MEM + Resid_LAN + Resid_EXF ≈ 0
```

This means the residual profile removes global severity by construction. As a result, the residuals should be interpreted as **relative profile scores**, not absolute impairment levels.

### 1.4 TopResidual and Gap

To assign a focal cognitive profile, two quantities are calculated:

```matlab
TopResidual = max([Resid_MEM, Resid_LAN, Resid_EXF]);
Gap = TopResidual - second_highest_residual;
```

These two values capture different aspects of profile strength:

```text
TopResidual: magnitude of the strongest relative domain impairment
Gap: separation between the strongest and second-strongest domain
```

A high TopResidual means one domain is meaningfully worse than the individual's average profile. A high Gap means the leading domain is clearly dominant rather than only slightly higher than the others.

### 1.5 Primary Subgroup Rule

The current primary rule is:

```text
TopResidual >= 0.50
and
Gap >= 0.20
```

If both conditions are satisfied, the observation is assigned to the domain with the highest residual:

```text
MEM_predominant
LAN_predominant
EXF_predominant
```

If the conditions are not satisfied, the observation is assigned to:

```text
No_focal_or_mixed
```

This label is not intended to represent a biological subtype. It means that the observation does not show a sufficiently clear focal domain-predominant pattern under the primary rule.

### 1.6 Why the Previous Multi-Domain Label Was Not Used

The earlier rule counted how many residual domains exceeded a threshold:

```text
0 elevated domains  -> No_domain
1 elevated domain   -> MEM/LAN/EXF predominant
>1 elevated domains -> Multi_domain
```

However, because the residuals are centered within person, `Multi_domain` in this framework does not mean global multidomain impairment. Instead, it usually means two domains are relatively higher than the individual's average and the third domain is relatively spared.

Thus, `Multi_domain` is better interpreted as a dual-pattern or non-focal profile, not as true multidomain disease severity. It was also too small at conservative thresholds for stable analysis. Therefore, the current method uses a single top-domain label plus `No_focal_or_mixed`.

### 1.7 Baseline Subgroup Distribution

Using the primary rule at baseline:

| Subgroup | Baseline N | Baseline % |
|---|---:|---:|
| No_focal_or_mixed | 847 | 35.48 |
| MEM_predominant | 964 | 40.39 |
| LAN_predominant | 217 | 9.09 |
| EXF_predominant | 359 | 15.04 |

## 2. Validation Experiments for the TopResidual/Gap Rule

Four validation checks were performed to evaluate whether the threshold rule is stable and not narrowly cherry-picked.

### 2.1 Threshold Sensitivity Grid

The primary threshold pair was evaluated against neighboring threshold combinations:

```text
TopResidual thresholds: 0.40, 0.50, 0.60, 0.70
Gap thresholds:         0.10, 0.20, 0.30
```

For each combination, subgroup counts and domain-profile direction were checked. The key question was whether the focal domain groups retained the expected profile direction:

```text
MEM_predominant group: highest mean Resid_MEM
LAN_predominant group: highest mean Resid_LAN
EXF_predominant group: highest mean Resid_EXF
```

Across all tested grid combinations, the expected domain direction was preserved. This suggests that the subgroup result is not dependent on a single narrow threshold pair.

Representative counts from the grid:

| TopResidual | Gap | No focal/mixed | MEM | LAN | EXF | Direction |
|---:|---:|---:|---:|---:|---:|---|
| 0.40 | 0.20 | 665 | 1056 | 265 | 401 | pass |
| 0.50 | 0.20 | 847 | 964 | 217 | 359 | pass |
| 0.60 | 0.20 | 1051 | 858 | 179 | 299 | pass |
| 0.70 | 0.20 | 1279 | 743 | 137 | 228 | pass |

Interpretation:

```text
Lower thresholds increase focal assignment but may include weaker profiles.
Higher thresholds increase specificity but reduce subgroup sample size.
The primary 0.50/0.20 threshold is a middle-ground operational choice.
```

### 2.2 Bootstrap Label Stability

A subject-level bootstrap was performed 500 times. In each bootstrap iteration:

```text
1. First-visit subjects were resampled.
2. CN/AP-negative reference mean and SD were recalculated.
3. Zimp, AvgZ, residuals, TopResidual, and Gap were recalculated.
4. Subgroup labels were reassigned using the primary 0.50/0.20 rule.
5. Reassigned labels were compared with the original labels.
```

Bootstrap results:

```text
Valid bootstrap iterations: 500
Mean simple agreement:      0.938
Mean Cohen's kappa:         0.909
```

Subgroup-level same-label probabilities:

| Original subgroup | N | Median same-label probability | IQR | % >= 0.70 | % >= 0.80 |
|---|---:|---:|---:|---:|---:|
| No_focal_or_mixed | 847 | 0.998 | 0.884-1.000 | 87.7 | 80.6 |
| MEM_predominant | 964 | 1.000 | 0.992-1.000 | 94.1 | 89.6 |
| LAN_predominant | 217 | 1.000 | 0.973-1.000 | 91.7 | 87.6 |
| EXF_predominant | 359 | 1.000 | 0.978-1.000 | 92.5 | 86.9 |

Interpretation:

```text
The labels are highly stable under uncertainty in reference scaling.
This supports the robustness of the subgroup rule.
```

### 2.3 Reference/Null Distribution Check

The CN/AP-negative reference group was used to evaluate whether the primary threshold behaves like a rare abnormality threshold.

Reference distribution:

| Distribution | TopResidual p50 | p80 | p90 | p95 | Gap p50 | p80 | p90 | p95 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| CN/AP-negative reference | 0.556 | 0.880 | 1.049 | 1.220 | 0.471 | 0.908 | 1.202 | 1.368 |
| All baseline | 0.672 | 1.088 | 1.342 | 1.548 | 0.615 | 1.236 | 1.646 | 1.937 |

The primary thresholds were positioned as follows in the CN/AP-negative reference group:

```text
TopResidual = 0.50: 43.1st percentile
Gap = 0.20:         23.3rd percentile
```

The percentage of CN/AP-negative reference subjects meeting both primary focal criteria was:

```text
51.9%
```

Important interpretation:

```text
The 0.50/0.20 rule should not be described as an abnormality threshold.
It is not rare among CN/AP-negative reference subjects.
It is better interpreted as an operational rule for relative cognitive profile assignment.
```

This is not necessarily a problem, because subgroup labels are intended to represent **relative cognitive phenotype**, not disease abnormality. However, reports should avoid claiming that focal subgroup labels imply pathological impairment.

### 2.4 Continuous Score Analysis

Hard subgroup labels were supplemented by continuous residual checks. Correlations were calculated between continuous profile variables and cognitive outcomes at baseline.

| Continuous variable | CDRSB | MMSE | MEM | LAN | EXF | AvgZ |
|---|---:|---:|---:|---:|---:|---:|
| Resid_MEM | 0.381 | -0.333 | -0.659 | 0.005 | 0.063 | 0.257 |
| Resid_LAN | -0.309 | 0.323 | 0.429 | -0.266 | 0.424 | -0.254 |
| Resid_EXF | -0.115 | 0.050 | 0.298 | 0.243 | -0.463 | -0.035 |
| TopResidual | 0.290 | -0.277 | -0.336 | 0.045 | 0.010 | 0.125 |
| Gap | 0.258 | -0.242 | -0.311 | 0.026 | 0.016 | 0.118 |

Interpretation:

```text
Higher Resid_MEM is associated with lower MEM score and worse global severity.
Higher Resid_LAN is associated with relatively lower LAN score but relatively better memory/global profile.
Higher Resid_EXF is associated with lower EXF score.
```

These analyses are sanity checks rather than independent validation because residuals are derived from the same cognitive domain scores.

### 2.5 Overall Validation Conclusion

The TopResidual/Gap method is appropriate as an operational relative-profile classifier because:

```text
1. It separates relative profile from global severity.
2. Threshold-grid sensitivity shows stable domain direction.
3. Bootstrap stability is strong.
4. The method avoids unstable Multi-domain labels.
```

The main caveat is:

```text
The threshold should not be interpreted as a normal-versus-abnormal cutoff.
It is a relative-profile assignment rule.
```

## 3. SILA estdtt0-Based Stage Definition

### 3.1 Analytical Goal

The purpose of stage definition is to represent **cognitive progression time**, not domain-specific phenotype. This is where SILA-derived `estdtt0` is useful.

In the current CDRSB-SILA model:

```text
estdtt0 = estimated years from CDRSB 0.5 threshold
```

Interpretation:

```text
estdtt0 < 0 : before estimated CDRSB onset
estdtt0 = 0 : estimated CDRSB 0.5 onset point
estdtt0 > 0 : years after estimated onset
```

This avoids a key problem of threshold-based cognitive staging: if observed cognitive scores fluctuate around a threshold, discrete stage labels can move backward over time. SILA instead maps observations onto a smooth longitudinal time axis, making progression time more stable.

Two stage-binning strategies were considered.

## 4. Stage Method 1: Fixed Time-Window Thresholds

### 4.1 Definition

The first method uses fixed time windows around the SILA onset anchor:

```text
Stage 0: estdtt0 < -5
Stage 1: -5 <= estdtt0 < 0
Stage 2: 0 <= estdtt0 < 3
Stage 3: 3 <= estdtt0 < 7
Stage 4: estdtt0 >= 7
```

Here, `0` is the only methodologically anchored value because it corresponds to estimated CDRSB 0.5 onset. The other boundaries are operational time-window cutpoints chosen to balance interpretability and sample size.

### 4.2 Stage-by-Subgroup Distribution

Using the current TopResidual/Gap subgroup rule:

| Stage | estdtt0 range | Total N | No_focal_or_mixed | MEM_predominant | LAN_predominant | EXF_predominant |
|---|---:|---:|---:|---:|---:|---:|
| 0 | < -5 | 2,319 | 985 (42.5%) | 458 (19.7%) | 403 (17.4%) | 473 (20.4%) |
| 1 | [-5, 0) | 1,931 | 837 (43.3%) | 480 (24.9%) | 270 (14.0%) | 344 (17.8%) |
| 2 | [0, 3) | 1,568 | 593 (37.8%) | 595 (37.9%) | 145 (9.2%) | 235 (15.0%) |
| 3 | [3, 7) | 2,704 | 673 (24.9%) | 1,676 (62.0%) | 85 (3.1%) | 270 (10.0%) |
| 4 | >= 7 | 1,094 | 192 (17.6%) | 780 (71.3%) | 10 (0.9%) | 112 (10.2%) |

### 4.3 Advantages

```text
1. Easy to interpret clinically because bins are expressed in years from onset.
2. The onset point, estdtt0 = 0, is preserved as a meaningful anchor.
3. The stages roughly follow CN -> MCI -> dementia severity gradients.
4. Useful for visualization and clinical narrative.
```

### 4.4 Limitations

```text
1. Boundaries such as -5, 3, and 7 years are not established clinical cutpoints.
2. Stage sizes are uneven.
3. Some subgroup-by-stage cells can become sparse.
4. The method may be criticized as using arbitrary time-window boundaries.
```

### 4.5 Interpretation

This method should be described as:

```text
onset-anchored operational time-window staging
```

It should not be described as a validated clinical staging system. It is useful when the goal is to communicate disease-time progression in interpretable year-based windows.

## 5. Stage Method 2: Pre-Onset Stage Plus Post-Onset Quantiles

### 5.1 Definition

The second method fixes all pre-onset observations as Stage 0 and divides post-onset observations into quartiles:

```text
Stage 0: estdtt0 < 0
Stage 1: 0 <= estdtt0 < post-onset 25th percentile
Stage 2: post-onset 25th percentile <= estdtt0 < post-onset 50th percentile
Stage 3: post-onset 50th percentile <= estdtt0 < post-onset 75th percentile
Stage 4: estdtt0 >= post-onset 75th percentile
```

In the current ADNI SILA output, the post-onset quartile cutpoints were:

```text
25th percentile: 2.62 years
50th percentile: 4.66 years
75th percentile: 6.61 years
```

Therefore:

```text
Stage 0: estdtt0 < 0
Stage 1: 0 <= estdtt0 < 2.62
Stage 2: 2.62 <= estdtt0 < 4.66
Stage 3: 4.66 <= estdtt0 < 6.61
Stage 4: estdtt0 >= 6.61
```

### 5.2 Stage-by-Subgroup Distribution

| Stage | estdtt0 range | Total N | No_focal_or_mixed | MEM_predominant | LAN_predominant | EXF_predominant |
|---|---:|---:|---:|---:|---:|---:|
| 0 | < 0 | 4,250 | 1,822 (42.9%) | 938 (22.1%) | 673 (15.8%) | 817 (19.2%) |
| 1 | [0, 2.62) | 1,341 | 516 (38.5%) | 493 (36.8%) | 131 (9.8%) | 201 (15.0%) |
| 2 | [2.62, 4.66) | 1,342 | 417 (31.1%) | 715 (53.3%) | 70 (5.2%) | 140 (10.4%) |
| 3 | [4.66, 6.61) | 1,341 | 293 (21.8%) | 885 (66.0%) | 24 (1.8%) | 139 (10.4%) |
| 4 | >= 6.61 | 1,342 | 232 (17.3%) | 958 (71.4%) | 15 (1.1%) | 137 (10.2%) |

### 5.3 Advantages

```text
1. Stage 0 has a clear onset-based interpretation: pre-onset.
2. Post-onset stages have balanced sample sizes by construction.
3. Balanced stage sizes reduce sparse subgroup-by-stage cells.
4. The method is less dependent on arbitrary year-based cutpoints.
5. It is useful for statistical comparison of subgroup distributions across progression time.
```

### 5.4 Limitations

```text
1. Cutpoints are cohort-specific and data-driven.
2. Values such as 2.62, 4.66, and 6.61 years are not clinical cutpoints.
3. Stage 0 combines all pre-onset observations, so early versus late pre-onset heterogeneity is not modeled.
4. External datasets may produce different post-onset quantile cutpoints.
```

### 5.5 Interpretation

This method should be described as:

```text
SILA onset-anchored post-onset quantile staging
```

It is best interpreted as a relative progression staging system within a cohort, not as an absolute clinical staging scale.

## 6. Comparison of the Two Stage Methods

| Feature | Fixed time-window method | Post-onset quantile method |
|---|---|---|
| Anchor | estdtt0 = 0 | estdtt0 = 0 |
| Boundary type | Operational year-based cutpoints | Data-driven post-onset quantiles |
| Interpretability | Stronger year-based clinical narrative | Stronger statistical balance |
| Sample size balance | Moderate/uneven | Better for Stage 1-4 |
| Risk of sparse cells | Higher | Lower |
| Clinical cutpoint validity | Not validated | Not validated |
| External validation meaning | Tests absolute time-window reproducibility | Tests relative progression-pattern reproducibility |
| Best use | Visualization/sensitivity | Primary subgroup-by-stage analysis |

## 7. Recommended Strategy

The recommended primary strategy is:

```text
Primary stage definition:
Stage 0 = estdtt0 < 0
Stages 1-4 = quartiles of estdtt0 among observations with estdtt0 >= 0
```

Rationale:

```text
1. The biologically/methodologically meaningful anchor is preserved at estdtt0 = 0.
2. Post-onset stages are balanced, improving subgroup-by-stage comparisons.
3. The method avoids arbitrary fixed year boundaries for post-onset progression.
4. It aligns well with the goal of comparing cognitive profile distributions across progression time.
```

The fixed time-window method can be used as a sensitivity analysis:

```text
Sensitivity stage definition:
Stage 0: estdtt0 < -5
Stage 1: -5 <= estdtt0 < 0
Stage 2: 0 <= estdtt0 < 3
Stage 3: 3 <= estdtt0 < 7
Stage 4: estdtt0 >= 7
```

This provides a more interpretable year-based progression summary.

## 8. External Validation Interpretation

If an external validation dataset is available, the recommended approach is:

```text
1. Independently fit SILA in the external dataset.
2. Estimate estdtt0 in the external dataset.
3. Define Stage 0 as estdtt0 < 0.
4. Divide post-onset estdtt0 >= 0 observations into quartiles within the external cohort.
5. Test whether subgroup-by-stage patterns are reproduced.
```

This validation tests whether the **relative progression pattern** is reproducible, not whether the exact numerical ADNI cutpoints are reproduced.

A useful sensitivity analysis is:

```text
Apply ADNI-derived cutpoints, 2.62, 4.66, and 6.61 years, directly to the external dataset.
```

This tests whether the pattern is also reproducible on an absolute SILA-time scale.

Recommended manuscript framing:

```text
Because absolute SILA time estimates may differ across cohorts due to follow-up duration, assessment schedules, and cohort composition, we used onset-anchored post-onset quantile stages within each cohort. External validation tested whether subgroup-by-stage patterns, rather than exact time cutpoints, were reproducible.
```

## 9. Final Interpretation

The current framework separates cognitive heterogeneity into two axes:

```text
Subgroup: relative cognitive profile based on TopResidual and Gap
Stage: cognitive progression time based on SILA estdtt0
```

This separation is important because it avoids conflating domain-specific cognitive phenotype with global severity or time since cognitive onset.

Recommended final terminology:

```text
Subgroup: residual-based cognitive profile subgroup
Stage: SILA-derived cognitive progression stage
```

Recommended primary analysis:

```text
Subgroup = TopResidual/Gap rule with MEM/LAN/EXF/No_focal_or_mixed labels
Stage = SILA onset-anchored post-onset quantile stages
```

Recommended sensitivity analysis:

```text
Stage = fixed onset-centered time windows
```
