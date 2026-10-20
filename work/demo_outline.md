# ML-11 case-study demo outline

- **Target length:** 5 minutes
- **Audience:** internship reviewers, hiring teams, or content and SEO practitioners
- **Live case study:** [Can a grouped model improve search-content review ranking?](https://divyam-chauhan.github.io/FL_Machine_learning/)

## Run of show

| Time | Show | Talk track |
|---|---|---|
| 0:00–0:40 | Paper title, question, and abstract | “I tested whether a March-feature ranking could prioritize human review better than the ML-07 March-impressions rule for a selected set of low-CTR pages.” State that the target is a retrospective April impression-decline proxy. |
| 0:40–1:20 | Data and cohort timing | Name warehouse build `v20260703` and table `fact_content_daily_performance`. Explain the 95,633 paired items and 39,545 ML-07 candidates. Call out that at least 20 GSC-available days in both months are required; April coverage defines cohort membership, so this exact cohort was not selectable at a March-time decision point. |
| 1:20–2:00 | Methodology and evaluation | Contrast L2 Logistic Regression with the specified rule on the same candidate pool. Explain the five client-grouped folds and that metrics are calculated inside each held-out fold because each fold has a separately fitted model. |
| 2:00–3:00 | P@50 chart and W07 results table | Lead with fold-macro P@50 of 0.804 vs. 0.420: a 0.384 absolute difference, about 1.9× the rule, with the model higher in every fold. Then note fold-macro ROC-AUC 0.636 vs. 0.438 and average precision 0.655 vs. 0.501. Identify this as a strong comparison against this particular rule on this selected retrospective cohort. |
| 3:00–3:35 | Fold design and separate W06 result | Point out that folds are uneven and two hold out one client each. If asked about W06, use its separate single-client-held-out diagnostic values; do not combine them with W07. |
| 3:35–4:20 | Limitations and recommendations | Explain that the target is not refresh success, the score is not calibrated confidence, and available fields do not establish page staleness. The ranking supports human review; it does not measure causal refresh impact, future-cohort performance, or traffic recovery. Briefly show the ranked aggregate review archetypes. |
| 4:20–5:00 | Reproducibility and close | Open the linked [Python 3.11 capstone notebook](https://github.com/Divyam-Chauhan/FL_Machine_learning/blob/main/work/notebooks/capstone.ipynb) and [aggregate report](https://github.com/Divyam-Chauhan/FL_Machine_learning/blob/main/work/capstone_report.md). Close with the decision supported: which eligible item a human might inspect first, not whether or how to edit it. |

## Demo safety

- Keep the demo on the deployed paper, aggregate charts and tables, and aggregate-only notebook outputs.
- Do not show or distribute action-queue rows, join IDs, URLs, titles, client names, domains, raw queries, or credentials. Do not open `.env` or expose a Hugging Face token.
- Do not describe the analysis as a live March queue or early-warning test. April availability is a cohort-selection limitation, not a model feature or label leakage through model inputs.
- Present W07 fold-local metrics as the main comparison. Do not pool scores from separately fitted folds or blend W06 and W07 results.

## Likely questions

**Does the score mean a page is stale or should be refreshed?** No. Page age and refresh timestamps are unavailable, and the score is not calibrated confidence or expected refresh lift.

**Does the result show that machine learning always beats rules?** No. It is a strong result against this specified ML-07 rule on one gate-selected retrospective cohort.

**Does the result show a refresh will recover traffic?** No. The observed April decline proxy is not a refresh outcome, and this analysis does not evaluate an intervention.
