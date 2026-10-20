# ML-12 demo outline

## ML-12 Week 8 showcase: 5-minute demo outline

**Case study:** [Live paper](https://divyam-chauhan.github.io/FL_Machine_learning/)
**Question:** For FlyRank's Lane 2 content-opportunity work, can a ranking help reviewers triage eligible high-impression, low-CTR pages more effectively than the ML-07 March-impressions rule?

- **0:00-0:45 | Question and decision.** Frame the practical problem: editors need to decide which eligible search-content items to inspect first for snippet fit, query-page fit, or internal discovery. The output is a review order, not an edit instruction.
- **0:45-1:45 | Method.** Explain the 39,545-item ML-07 pool, the March-only L2 Logistic Regression, the March-impressions rule, and five client-grouped folds. Metrics are computed within each held-out fold because each fold uses a separately fitted model. Mention that April coverage helps define the evaluated cohort, so it was not selectable at a March-time decision point.
- **1:45-2:30 | One chart.** Show `work/figures/capstone_fold_precision_at_50.png`. Read the five fold-local P@50 pairs and point out that the model is higher in every fold; do not display or share any action-queue rows.
- **2:30-3:40 | One honest result.** Fold-macro P@50 is 0.804 for the model vs. 0.420 for this rule, a 0.384 absolute difference (about 1.9x). This is a strong result against this particular rule on one selected retrospective cohort. The folds are uneven, including two that hold out one client each; the metrics are descriptive and do not establish future-cohort performance or refresh impact.
- **3:40-4:35 | One recommendation.** First verify measurement coverage for the 1,201 partial-observation candidates before treating them as content opportunities or editing. For other candidates, inspect the current page and query context before deciding whether a change is appropriate.
- **4:35-5:00 | Close.** Re-state the human-review use and link the [paper](https://divyam-chauhan.github.io/FL_Machine_learning/) and [executed capstone notebook](https://github.com/Divyam-Chauhan/FL_Machine_learning/blob/main/work/notebooks/capstone.ipynb).

**Demo safety:** Keep the presentation to the public paper and aggregate charts/tables. Never show queue rows, join IDs, URLs, titles, client names, domains, raw queries, or credentials. Do not call this a live March queue or an early-warning test; do not interpret the score as calibrated confidence, page staleness, or expected refresh lift.
