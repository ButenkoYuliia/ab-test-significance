# A/B Test Statistical Significance

Statistical analysis of four A/B tests of an e-commerce website: a Python script checks whether changes in key conversion metrics are statistically significant, and a Tableau dashboard shows where the metrics really changed and where they did not.

![Significance dashboard](images/dashboard_significance.png)

## Links

- 📊 **Tableau dashboard:** [A/B Test Dashboard (Overview + Significance)](https://public.tableau.com/views/ABTestDashboard_17925770139850/Significance)
- 📓 **Google Colab notebook (Ukrainian):** [ab_test_significance_ua.ipynb](https://colab.research.google.com/github/ButenkoYuliia/ab-test-significance/blob/main/ab_test_significance_ua.ipynb)
- 📁 **Final results (totals per test):** [data/ab_test_results_final.csv](data/ab_test_results_final.csv)
- 📁 **All segments (used in Tableau):** [data/ab_test_results_segments.csv](data/ab_test_results_segments.csv)

## Business question

Which of the four A/B tests produced a real (statistically significant) change in the checkout funnel and sign-ups, and which differences are just random noise?

## Data

- Source: Google BigQuery, e-commerce dataset "DA" (GA4-style events), Nov 2020 – Jan 2021; the notebook queries BigQuery directly
- Aggregated by date, test, test group, device, continent, country and channel
- Numerator = **number of events**; denominator = **number of sessions**

## Metrics

| Metric | Formula |
|---|---|
| add_payment_info / session | `add_payment_info` events ÷ sessions |
| add_shipping_info / session | `add_shipping_info` events ÷ sessions |
| begin_checkout / session | `begin_checkout` events ÷ sessions |
| new_accounts / session | new accounts ÷ sessions |

## Method

1. Metrics of the control group (`test_group = 1`) and the test group (`test_group = 2`) are compared with a **two-proportion z-test** (`statsmodels.proportions_ztest`).
2. Significance level **α = 0.05**: p-value < 0.05 → significant increase or decrease; otherwise not significant.
3. `metric_change` = (test − control) / control × 100 %.
4. The calculation runs in **loops** over all tests, metrics and segments (total, device, continent, country, channel). Only the metric names and formulas are hard-coded, so new metrics or segments can be added with one line.
5. Segments with fewer than 1,000 sessions per group are flagged as unreliable.

## Key findings

| Test | Metric | Control | Test | Change | p-value | Result |
|---|---|---|---|---|---|---|
| 1 | add_payment_info / session | 4.38 % | 4.93 % | +12.5 % | 0.0001 | ✅ significant increase |
| 1 | add_shipping_info / session | 6.69 % | 7.13 % | +6.6 % | 0.009 | ✅ significant increase |
| 1 | begin_checkout / session | 8.34 % | 8.90 % | +6.7 % | 0.003 | ✅ significant increase |
| 3 | begin_checkout / session | 13.61 % | 13.15 % | −3.4 % | 0.012 | 🔻 significant decrease |
| 4 | begin_checkout / session | 11.95 % | 11.67 % | −2.4 % | 0.046 | 🔻 significant decrease |
| 4 | new_accounts / session | 8.55 % | 8.26 % | −3.4 % | 0.018 | 🔻 significant decrease |
| other 10 checks | | | | −3.5 % to +3.6 % | > 0.05 | not significant |

- **Test 1** significantly improves the whole checkout funnel, with the largest gain at the payment step (+12.5 %). Recommended for rollout.
- **Test 3** significantly reduces `begin_checkout`. Not recommended.
- **Test 4** significantly reduces `begin_checkout` and sign-ups. Not recommended.
- **Test 2** shows no significant effect.

**Limitation:** one session can contain several events, while the z-test assumes independent observations, so significance may be slightly overstated. Isolated significant segments without confirmation in the total are treated as hypotheses.

## Dashboard

Two pages on Tableau Public:

- **Overview:** group split, distribution of groups by device, country, channel and continent, event counts with % difference
- **Significance:** uplift per metric coloured by significance, conversion A vs B, results table with p-values, explanation of the method; filter by test number

## Tools

Python (pandas, NumPy, statsmodels) · Google Colab · Google BigQuery (SQL) · Tableau Public

## Repository structure

```
├── ab_test_significance_ua.ipynb  # analysis notebook (Ukrainian)
├── data/
│   ├── ab_test_results_final.csv    # totals per test
│   └── ab_test_results_segments.csv # all segments, used in Tableau
├── images/
│   └── dashboard_significance.png
└── README.md
```

## Author

**Yuliia Butenko** · Data Analyst · [GitHub](https://github.com/ButenkoYuliia)
