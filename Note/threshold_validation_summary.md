# TopResidual/Gap Threshold Validation Summary

Generated: 2026-05-01 06:53:19

## Analysis Rule

- Reference: first-visit CN/AP-negative subjects (`n = 262`).
- Baseline subjects: `n = 2387`.
- Impairment score: `Zimp = -(raw cognition - reference mean) / reference SD`.
- Relative profile: `Resid_domain = Zimp_domain - mean(Zimp_MEM, Zimp_LAN, Zimp_EXF)`.
- Primary focal rule: `TopResidual >= 0.50` and `Gap >= 0.20`; label is assigned by the top residual domain.
- Stage variables and the previous `Multi_domain` label were not used.

## Primary Threshold Counts

| Subgroup | Baseline N | Baseline % |
|---|---:|---:|
| No_focal_or_mixed | 847 | 35.48 |
| MEM_predominant | 964 | 40.39 |
| LAN_predominant | 217 | 9.09 |
| EXF_predominant | 359 | 15.04 |

## 1. Threshold Sensitivity Grid

| TopResidual threshold | Gap threshold | No focal/mixed | MEM | LAN | EXF | Direction check |
|---:|---:|---:|---:|---:|---:|---|
| 0.40 | 0.10 | 597 (25.0%) | 1080 (45.2%) | 284 (11.9%) | 426 (17.8%) | pass |
| 0.40 | 0.20 | 665 (27.9%) | 1056 (44.2%) | 265 (11.1%) | 401 (16.8%) | pass |
| 0.40 | 0.30 | 767 (32.1%) | 997 (41.8%) | 243 (10.2%) | 380 (15.9%) | pass |
| 0.50 | 0.10 | 802 (33.6%) | 981 (41.1%) | 228 (9.6%) | 376 (15.8%) | pass |
| 0.50 | 0.20 | 847 (35.5%) | 964 (40.4%) | 217 (9.1%) | 359 (15.0%) | pass |
| 0.50 | 0.30 | 910 (38.1%) | 928 (38.9%) | 206 (8.6%) | 343 (14.4%) | pass |
| 0.60 | 0.10 | 1028 (43.1%) | 869 (36.4%) | 182 (7.6%) | 308 (12.9%) | pass |
| 0.60 | 0.20 | 1051 (44.0%) | 858 (35.9%) | 179 (7.5%) | 299 (12.5%) | pass |
| 0.60 | 0.30 | 1089 (45.6%) | 836 (35.0%) | 173 (7.2%) | 289 (12.1%) | pass |
| 0.70 | 0.10 | 1266 (53.0%) | 748 (31.3%) | 139 (5.8%) | 234 (9.8%) | pass |
| 0.70 | 0.20 | 1279 (53.6%) | 743 (31.1%) | 137 (5.7%) | 228 (9.6%) | pass |
| 0.70 | 0.30 | 1299 (54.4%) | 732 (30.7%) | 134 (5.6%) | 222 (9.3%) | pass |

Primary and neighboring thresholds preserved the expected focal-domain direction when each focal subgroup had nonzero membership. This supports that the primary result is not unique to one narrow threshold pair.

## 2. Bootstrap Label Stability

- Bootstrap iterations requested: `500`; valid iterations: `500`.
- Mean simple agreement across valid iterations: `0.938` (SD `0.026`).
- Mean Cohen kappa across valid iterations: `0.909` (SD `0.038`).

| Original subgroup | N | Median same-label probability | IQR | % >= 0.70 | % >= 0.80 |
|---|---:|---:|---:|---:|---:|
| No_focal_or_mixed | 847 | 0.998 | 0.884-1.000 | 87.7 | 80.6 |
| MEM_predominant | 964 | 1.000 | 0.992-1.000 | 94.1 | 89.6 |
| LAN_predominant | 217 | 1.000 | 0.973-1.000 | 91.7 | 87.6 |
| EXF_predominant | 359 | 1.000 | 0.978-1.000 | 92.5 | 86.9 |

Bootstrap recalculated the CN/AP-negative reference distribution each time and reassigned labels on the original baseline subjects. This mainly tests robustness to reference-scaling uncertainty.

## 3. Reference/Null Distribution

| Distribution | TopResidual p50 | p80 | p90 | p95 | Gap p50 | p80 | p90 | p95 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| CN/AP-negative reference | 0.556 | 0.880 | 1.049 | 1.220 | 0.471 | 0.908 | 1.202 | 1.368 |
| All baseline | 0.672 | 1.088 | 1.342 | 1.548 | 0.615 | 1.236 | 1.646 | 1.937 |

- `TopResidual = 0.50` is at the 43.1th percentile of the CN/AP-negative reference distribution.
- `Gap = 0.20` is at the 23.3th percentile of the CN/AP-negative reference distribution.
- Reference subjects meeting both primary focal criteria: 51.9%.
- All baseline subjects meeting both primary focal criteria: 64.5%.

## 4. Continuous Score Analysis

Correlations are Pearson correlations across baseline subjects. These are sanity checks for continuous profile consistency, not independent validation because residuals are derived from the same domain scores.

| Continuous variable | CDRSB | MMSE | MEM | LAN | EXF | AvgZ |
|---|---:|---:|---:|---:|---:|---:|
| Resid_MEM | 0.381 | -0.333 | -0.659 | 0.005 | 0.063 | 0.257 |
| Resid_LAN | -0.309 | 0.323 | 0.429 | -0.266 | 0.424 | -0.254 |
| Resid_EXF | -0.115 | 0.050 | 0.298 | 0.243 | -0.463 | -0.035 |
| TopResidual | 0.290 | -0.277 | -0.336 | 0.045 | 0.010 | 0.125 |
| Gap | 0.258 | -0.242 | -0.311 | 0.026 | 0.016 | 0.118 |

Regression sanity checks: raw domain score was modeled as `AvgZ + matching domain residual`.

| Outcome | AvgZ beta | AvgZ p | Matching residual beta | Residual p | Adjusted R2 |
|---|---:|---:|---:|---:|---:|
| MEM | -0.492 | 0 | -0.492 | 0 | 1.000 |
| LAN | -0.552 | 0 | -0.552 | 0 | 1.000 |
| EXF | -0.544 | 0 | -0.544 | 0 | 1.000 |

## Conclusion

- The `TopResidual >= 0.50` and `Gap >= 0.20` rule gives a clear focal-profile definition while keeping `No_focal_or_mixed` as a non-biological comparison label.
- Sensitivity-grid results should be interpreted as the main protection against cherry-picking: neighboring threshold pairs preserve the expected domain-profile direction.
- Bootstrap agreement/kappa indicate `strong` stability under reference-scaling uncertainty.
- The reference/null distribution is the main caution: `TopResidual = 0.50` and `Gap = 0.20` are not rare within CN/AP-negative reference subjects, so this rule should not be described as an abnormality threshold.
- The threshold is reasonable as an operational relative-profile rule, not as a normal-versus-impaired cutoff. Downstream reports should emphasize robustness across the grid and avoid claiming that focal labels are absent in cognitively normal/AP-negative subjects.
