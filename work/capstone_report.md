# Prioritizing Content Refresh Reviews with Search and Engagement Signals

- **Author:** Capstone project
- **Lane:** Applied search intelligence / content refresh prioritization
- **Date:** 2026-10-16

## 1. Problem framing

This project asks whether a learned ranking can help editors identify existing content that is more likely to show declining search traffic. The decision is which pages to review first when editorial time is limited. The output is a ranked, human-reviewed refresh queue; an editor decides whether to inspect a page, revise its content, review its search snippet, or leave it alone. A false positive wastes review time; a false negative may leave a useful page's decline unnoticed. The model does not prescribe automatic publishing.

## 2. Data safety

The analysis uses the repository's 30,000-row anonymized starter snapshot, with 32 pseudonymized client groups. It filters to records with positive 90-day impressions and content age of at least 90 days. The binary label is `trend_direction == "down"`. `trend_direction` and `trend_pct` are not features. `client_id` is used only for the group split; `content_id` is not a feature. The model also excludes last-30-day fields. No URLs, titles, raw queries, client names, or row-level recommendations are included in the published paper.

## 3. Baseline

The transparent heuristic uses 90-day impressions, time since last update, and average search position:

`100 × (0.40 × visibility + 0.30 × freshness + 0.25 × position opportunity)`

Visibility is `log1p(impressions_90d) / log1p(30000)`. Freshness is `days_since_last_update / 180`, clipped to `[0, 1]`. Position opportunity is 1.0 for positions 4–20, 0.5 for positions 1–3, and 0.1 for other or unavailable positions. The position bands are mutually exclusive; the notebook previously tested the broad 1–20 band before the nested 1–3 band, making the latter unreachable. That logic is corrected in the reported results.

## 4. Model / analysis

The target is a historical decline indicator, not a future outcome: `is_declining_label = (trend_direction == "down")`. The analysis compares the heuristic with a 100-tree Random Forest (`max_depth=10`, `random_state=42`). Inputs are 18 numeric signals (`search_volume`, `competition`, `cpc`, `word_count`, `char_count`, log-transformed 90-day impressions/clicks/sessions/AI sessions, days with impressions/sessions, content age, days since update, CTR, average position, engagement rate, scroll rate, and AI traffic percentage) and five categorical fields (`competition_level`, `content_type`, `main_intent`, `age_tier`, `position_tier`). One-hot encoding produces 52 model columns for this run.

## 5. Evaluation

`GroupShuffleSplit(test_size=0.20, random_state=42)` holds out complete client groups: 25 training clients (23,837 rows) and 7 unseen test clients (6,163 rows). The test-set decline prevalence is 0.511. The comparison below uses the same test rows and the same top-K definition for both rankers.

| Ranker | Precision@20 | Precision@50 | ROC-AUC |
|---|---:|---:|---:|
| Heuristic baseline | 0.200 | 0.340 | 0.529 |
| Random Forest | 0.450 | 0.580 | 0.609 |

At 50 reviewed pages, the model's precision is 0.240 higher than the heuristic and about 1.7 times its precision. Precision@50 is 0.069 above the test-set prevalence; precision@20 (0.450) is below prevalence. ROC-AUC is only 0.609, so overall separation remains modest. These top-K results are directional evidence from one client holdout, not a guarantee of performance on new samples.

These are capstone-notebook results, not a rerun of the reference pipeline benchmark in `outputs/model_report.md`. The reference pipeline uses a different client split, baseline, feature encoding, and Random Forest configuration (200 trees and `min_samples_leaf=25`, versus 100 trees and no class weighting here); its reported 0.680/0.240 Precision@50 values must not be mixed into this table.

## 6. Interpretation

The largest Random Forest impurity importances were days with impressions (0.1582), log 90-day impressions (0.1160), average position (0.1109), and content age (0.0916), followed by word count (0.0643), character count (0.0521), CTR (0.0388), and scroll rate (0.0364). These describe how the fitted trees split this dataset; they are not causal effects, and correlated signals can share or distort importance.

The two figures embedded in the deployed page show the corrected same-holdout Precision@50 comparison and these top eight relative importances.

## 7. Recommendation

Use the ranked list only as a decision-support queue. Editors should verify the underlying page and context before acting, then record what was changed and monitor outcomes. The full-data action summary from this run was: refresh 10,877; CTR review 9,428; monitor 6,428; engagement review 3,204; expand and refresh 63. These are rule-generated aggregate counts, not validated outcomes. The model was fitted on the training-client subset before producing the exploratory full-portfolio queue; those queue scores are not all out-of-sample. Do not publish row-level queue exports or pseudonymous IDs.

## 8. Reproducibility

Verified with Python 3.11.9, NumPy 2.4.6, pandas 3.0.6, scikit-learn 1.9.1, and Matplotlib 3.10.8. The random seed is 42 for both the group split and forest.

From a fresh clone, install the project requirements and Jupyter, open the notebook, and use **Run All**:

```bash
python -m pip install -r requirements.txt jupyter
jupyter notebook work/notebooks/capstone.ipynb
```

Run from the repository root or the notebook directory. The input dataset is the bundled `data/raw/content_refresh_anonymized.csv`; generated capstone figures belong in `work/outputs/`. The deployed page uses copies of the aggregate figures in `docs/`.

---

**Claim boundary:** This is an observational ranking exercise on an anonymized starter snapshot. It measures alignment with a historical decline label; it does not predict Google's ranking algorithm, establish why traffic changed, or estimate the causal impact of refreshing content.
