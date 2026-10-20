# ML-11 Capstone — Refresh / Content Opportunity Scoring

- **Author:** Divyam Chauhan
- **Lane:** Lane 2 — Refresh / Content Opportunity Scoring
- **Source:** Gated Hugging Face `FlyRank/internship-warehouse`, release/build `v20260703`, table `fact_content_daily_performance`
- **Feature window / outcome window:** March / April 2026
- **Execution:** Python 3.11.9, seed 42

> This is a retrospective ranking of an observed impression-decline proxy among ML-07-eligible items. It supports human review only; it does not establish early-warning ability, causal impact, or traffic recovery from a refresh. The evaluated cohort is not a live March queue or an early-warning test.

## 1. Problem framing

The decision is which eligible page-client items an editor should inspect first. The analysis unit is one page-client pair after monthly aggregation. It compares a March-only machine-learning ranking with a transparent March-impressions rule. A reviewer must inspect the current page, query context, data coverage, and business value before deciding whether to change anything.

## 2. Data safety and contract

The `fact_content_daily_performance` daily fact table from release/build `v20260703` is aggregated to one page-client pair per month. Eligibility for the paired frame requires at least 20 days with `gsc_data_available IS TRUE` in both March and April 2026 and positive March impressions. April availability is not a model feature, but using it to define cohort membership means this exact cohort could not have been selected at a March-time decision point. This cohort-selection/decision-time limitation is not label leakage. The 20-day cutoff requires substantial GSC observation in both months, while positive March impressions provide the nonzero baseline for the 80%-of-March comparison; pairs failing these criteria are outside this defined cohort. The resulting frame has 95,633 pairs; 47,323 (49.5%) meet the April decline proxy before candidate selection.

The outcome is April impressions at most 80% of March impressions. It is a retrospective next-month proxy, not a refresh-success label. The ML-07 gate requires March impressions >=500, average position >0 and <=20, and CTR <0.5%. The gate selects 39,545 candidates across 31 client groups; the proxy rate in this pool is 55.2%. Results are conditional on this selected pool.

Client/content IDs are used only for joins and grouping, never as features. April values and label-derived fields are not predictors. The notebook displays aggregate results only and does not preview or export the row-level queue. The available features contain no page age or refresh timestamp, so this work cannot identify stale pages.

## 3. Baseline

The ML-07 rule ranks eligible candidates by `log1p(March impressions)`, with March clicks and average position as tie-breakers. This is a transparent comparison on the same eligible population and outcome used for the model.

## 4. Model and evaluation

The model is L2-regularized Logistic Regression with median imputation and standard scaling fit within each training fold. Its five March-only inputs are log impressions, log clicks, CTR, average position, and GSC-available days. Client and content IDs are grouping/join keys only.

W07 uses five-fold GroupKFold by client. Each validation fold is scored by a model trained on the other client groups. Since every fold fits a separate imputer, scaler, and logistic model, pooled raw scores are not assumed comparable across folds. The reported W07 comparisons are therefore calculated within each held-out fold. Two folds hold out one client each; the folds vary substantially in clients and item counts.

W06 is a separate single client-held-out diagnostic. Its metrics are reported separately and not blended with W07.

## 5. Results

The candidate-pool proxy rate is 0.552. The table reports within-fold W07 results; P@50, ROC-AUC, and average precision are shown for both rankings.

| Fold | Held-out clients | Items | Base rate | Model P@50 | Rule P@50 | Model AUC | Rule AUC | Model AP | Rule AP |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 1 | 1 | 11,069 | 0.484 | 0.860 | 0.180 | 0.603 | 0.448 | 0.583 | 0.438 |
| 2 | 1 | 9,435 | 0.733 | 0.920 | 0.520 | 0.652 | 0.447 | 0.819 | 0.695 |
| 3 | 6 | 6,384 | 0.420 | 0.720 | 0.300 | 0.632 | 0.444 | 0.544 | 0.375 |
| 4 | 7 | 6,347 | 0.547 | 0.600 | 0.500 | 0.606 | 0.490 | 0.622 | 0.536 |
| 5 | 16 | 6,310 | 0.543 | 0.920 | 0.600 | 0.687 | 0.359 | 0.706 | 0.464 |

Across-fold macro means, computed from unrounded fold metrics, were: P@10 0.780 vs 0.500; P@20 0.800 vs 0.460; P@50 0.804 vs 0.420; ROC-AUC 0.636 vs 0.438; and average precision 0.655 vs 0.501 (model vs rule). The fold table rounds each value to three decimals, so recomputing means from its displayed values may differ in the last decimal. These descriptive means are not confidence intervals. W07's existing pooled receipt is numerically reproducible, but its ranking metrics compare scores from separately fitted models; pooled metrics are not used as the headline.

W06, separately: on 19,084 held-out candidates with a 0.554 base rate, model P@10/P@20/P@50 was 0.80/0.75/0.72 versus 0.40/0.30/0.32 for the rule. ROC-AUC was 0.448 for both and average precision was 0.536 for both. This is a split-specific top-of-queue result, not a broad ranking improvement.

## 6. Interpretation

Within the selected pool, the model ranked proxy-positive items higher than the rule within each W07 fold at P@50. The fold rates and sizes vary, and the evaluation covers only one adjacent-month cohort. The score is not calibrated confidence or expected refresh lift. No causal, early-warning, or future-cohort claim follows from these results.

## 7. Ranked recommendations

W07's rule-based archetypes suggest a human review order:

1. **Verify coverage before editing:** 1,201 partial-observation candidates; confirm page existence and measurement coverage.
2. **Review snippet and search intent:** 29,932 page-one low-CTR candidates; inspect query mix, result features, snippet fit, and factual accuracy.
3. **Review query-page fit and internal links:** 8,412 striking-distance low-CTR candidates; check intent alignment and internal discovery context.

These counts describe routing rules, not validated actions or outcomes. Do not publish, delete, redirect, canonicalize, merge, or noindex a page from a score alone.

## 8. Reproducibility

Run `work/notebooks/capstone.ipynb` from the repository root using Python 3.11.9 and a Python 3.11 Jupyter kernel. The notebook reads March/April partitions directly from the gated warehouse; the Hugging Face token is read from the ignored `.env` and never printed. Seed: 42. Verified package versions: NumPy 2.4.6, pandas 3.0.6, scikit-learn 1.9.1, DuckDB 1.5.6.

Aggregate figures are saved under `work/figures/`. Generated queue CSVs remain ignored by Git and are not linked or included here.

## 9. Acknowledgments and data credit

Thanks to [FlyRank](https://flyrank.ai/) for the internship program and warehouse access. The assigned [March 2026 FlyRank SEO research paper](../docs/flyrank-seo-research-march-2026.pdf) is background context only; its portfolio results are not findings from this model.
