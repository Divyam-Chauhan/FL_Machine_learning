# Prioritizing Content Refresh Reviews with Search and Engagement Signals

- **Author:** Capstone project
- **Lane:** Applied search intelligence / content refresh prioritization
- **Date:** 2026-10-16

## 1. Problem framing

This project compares a trained model with a hand-written scoring rule to see how they rank records carrying a historical search-impression decline label. The model is a Random Forest: it combines many decision trees, each of which applies a sequence of yes/no splits to the input information. The score could help an editor decide which pages to inspect first, but does not decide whether to make a change. A high-ranked record without the decline label may use review time unnecessarily; a decline-labelled record ranked lower may not be reviewed.

## 2. Data safety

The analysis uses the repository's 30,000-row anonymized starter snapshot, with 32 pseudonymized client groups. It filters to records with positive 90-day impressions and content age of at least 90 days. The binary label is `trend_direction == "down"`: impressions in the latest 30-day period are more than 20% below the preceding 30-day period. This records an already-observed change, not a future outcome. `trend_direction` and `trend_pct` are not model inputs. `client_id` is used only for the group split; `content_id` is not a feature. However, the 90-day input metrics overlap the 30-day windows used to form the label. The evaluation therefore measures retrospective ranking alignment on held-out clients, not advance warning. No URLs, titles, raw queries, client names, or row-level recommendations are included in the published paper.

## 3. Baseline

The transparent hand-written scoring rule uses three quantities: 90-day search impressions (as a visibility measure), days since last update (as a freshness measure), and average search position (as a visibility/opportunity measure):

`100 × (0.40 × visibility + 0.30 × freshness + 0.25 × position opportunity)`

Visibility is `log1p(impressions_90d) / log1p(30000)`. Freshness is `days_since_last_update / 180`, clipped to `[0, 1]`. Position opportunity is 1.0 for positions 4–20, 0.5 for positions 1–3, and 0.1 for other or unavailable positions. The position bands are mutually exclusive; the notebook previously tested the broad 1–20 band before the nested 1–3 band, making the latter unreachable. That logic is corrected in the reported results.

## 4. Model / analysis

The target is an already-observed decline indicator, not a future outcome: `is_declining_label = (trend_direction == "down")`. The analysis compares the rule with a Random Forest that combines 100 decision trees (`max_depth=10`, `random_state=42`). Inputs are 18 numeric signals (`search_volume`, `competition`, `cpc`, `word_count`, `char_count`, log-transformed 90-day impressions/clicks/sessions/AI sessions, days with impressions/sessions, content age, days since update, CTR, average position, engagement rate, scroll rate, and AI traffic percentage) and five categorical fields (`competition_level`, `content_type`, `main_intent`, `age_tier`, `position_tier`). The five categories are converted into numeric indicator columns, producing 52 input columns for this run.

## 5. Evaluation

`GroupShuffleSplit(test_size=0.20, random_state=42)` holds out complete client groups: 25 training clients (23,837 rows) and 7 unseen test clients (6,163 rows). In the test group, 51.1% of records carried the decline label. The comparison below uses the same test rows and top-20/top-50 cutoffs for both methods. The top-20 and top-50 shares are the fractions of those records carrying the label. ROC-AUC summarizes how often a decline-labelled record is ranked above a non-decline record across cutoffs; 0.5 is chance-level ordering.

| Scoring approach | Share with label (top 20) | Share with label (top 50) | Overall ranking score (ROC-AUC) |
|---|---:|---:|---:|
| Hand-written rule | 20.0% | 34.0% | 0.529 |
| Trained model | 45.0% | 58.0% | 0.609 |

In this test set, 29 of the model's top 50 records had the decline label, compared with 17 of the rule's top 50. That is a 24-percentage-point difference in the share for this one test split. The model's top-50 share is 6.9 percentage points above the 51.1% test-set share; its top-20 share (45.0%) is below that level. The overall ranking score of 0.609 indicates only modest separation here. These are retrospective results from one client holdout, not evidence of advance warning or a guarantee of performance on new samples.

These are capstone-notebook results, not a rerun of the reference pipeline benchmark in `outputs/model_report.md`. The reference pipeline uses a different client split, baseline, feature encoding, and Random Forest configuration (200 trees and `min_samples_leaf=25`, versus 100 trees and no class weighting here); its reported 0.680/0.240 Precision@50 values must not be mixed into this table.

## 6. Interpretation

The largest tree-split importance scores were days with impressions (0.1582), log 90-day impressions (0.1160), average position (0.1109), and content age (0.0916), followed by word count (0.0643), character count (0.0521), CTR (0.0388), and scroll rate (0.0364). These scores describe how the fitted trees used the inputs to separate the labels; they are not causal effects, and related inputs can share or distort importance.

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
