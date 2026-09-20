# Can Search Signals Prioritize Content Refresh Reviews?

**Batuhan Şahin · Refresh / Content Opportunity Scoring · September 2026**

## 0. Abstract

This study asks whether pre-outcome search and content signals can prioritize pages for human refresh review. Using a public-safe FlyRank warehouse release, I created 101,280 page-month observations across 40 client groups, using March 2025 features and an April 2025 outcome. A shallow decision tree was compared with a transparent hand rule on the same grouped holdout, with client-disjoint splits, base rates, leakage checks, and Precision@K as the operating metric. On the grouped test set, the tree improved Precision@50 from 0.26 to 0.38 and ROC-AUC from 0.467 to 0.570, but Precision@50 remained below the test base rate of 0.558 and validation performance was weak. Because the evidence is directional and unstable across groups, the deployed recommendation is a rule-based queue of 176,738 pages with reason codes and mandatory human review rather than automated content changes.

## 1. Introduction / Problem statement

Editorial teams can inspect only a fraction of a large content inventory. This work supports the decision of which pages a reviewer should inspect first. It tests whether signals available before an outcome window—visibility, click-through rate, observed data coverage, average position, and content age—can support that triage decision.

The target is intentionally narrow: rank candidates for human review. The analysis does not estimate the causal impact of refreshing a page, prove how Google’s algorithm works, or prescribe automatic publishing, merging, pruning, deletion, or redirects.

## 2. Data

The analysis uses the full gated FlyRank warehouse release referenced by the internship materials. Features came from March 2025 page-level Google Search Console aggregates joined to safe content metadata; the outcome came from April 2025. Only hashed identifiers were used for joins and group assignment.

The modeling set contains 101,280 rows across 40 client groups. Pages required usable March feature values and at least 20 observed April GSC days. Rows without sufficient future-window coverage, missing join keys, or safe feature values were excluded. The action queue uses pre-outcome page signals and covers 176,738 pages, a broader set than the outcome-eligible modeling population.

Requiring at least 20 April observed days improves label reliability but creates a selected evaluation population. Results may not transfer to sparse or newly observed pages.

## 3. Methodology

The label is whether April clicks exceeded March clicks for an outcome-eligible page. Every model feature was available in March or came from static content metadata.

- Features: March CTR, log impressions, average position, observed March days, and content age.
- Excluded from features: identifiers, client labels, product flags, April fields, and label-derived values.
- Baseline: a hand rule prioritizing pages at least 180 days old with at least 500 March impressions.
- Model: decision-tree classifier, with depth selected from 2–5 by validation average precision. Depth 2 was selected with class balancing and random seed 42.
- Validation: client-disjoint train, validation, and test sets with 24, 8, and 8 groups. Test data remained untouched until the final comparison.
- Metrics: Precision@20, Precision@50, average precision, ROC-AUC, and the population base rate.

## 4. Results

| Evaluation | Rows | Base rate | P@20 | P@50 | Avg. precision | ROC-AUC |
|---|---:|---:|---:|---:|---:|---:|
| Validation — depth-2 tree | 9,135 | 0.339 | 0.100 | 0.060 | 0.327 | 0.472 |
| Grouped test — hand rule | 46,850 | 0.558 | 0.250 | 0.260 | 0.525 | 0.467 |
| Grouped test — depth-2 tree | 46,850 | 0.558 | 0.300 | 0.380 | 0.583 | 0.570 |

The model improved every reported test ranking/discrimination metric over the hand rule on the same 46,850 rows. However, its 0.38 Precision@50 remained below the 0.558 test base rate, and validation Precision@50 was 0.06 against a 0.339 validation base rate. The audit verdict is therefore **LIMITED**: the model beats the hand rule on the grouped test but is unstable across groups and does not justify automation.

In the fitted depth-2 tree, observed March days had importance 0.501 and March CTR had importance 0.499; the remaining three features had zero importance. These impurity-based values describe this fitted tree only and are not causal evidence or proof of ranking factors.

## 5. Limitations & honest framing

- Validation and grouped test outcomes differ, indicating instability across client groups.
- The future-coverage requirement may overrepresent mature, well-observed pages.
- Click growth is a proxy, not a direct measure of refresh need, content quality, or incremental impact.
- Seasonality, intent changes, SERP features, business value, and editorial constraints are not fully represented.
- Feature importance reflects split usage in one fitted model; it is not a causal or algorithmic claim.

The correct framing is **observed, directional, and decision-support**.

## 6. Ranked recommendations

Because the model audit was limited, the operating queue uses a transparent rule plus mandatory human review. A page qualifies at age ≥180 days and March impressions ≥500, with bonuses for age ≥365 days and impressions ≥5,000. Average position separates `REVIEW_NEXT` from `MONITOR` at the score-4 boundary.

| Rank | Action | Pages | Share | Recommended use |
|---:|---|---:|---:|---|
| 1 | REVIEW_NOW | 13,154 | 7.44% | Check freshness, intent, cannibalization, and business importance; choose any editorial action manually. |
| 2 | REVIEW_NEXT | 15,435 | 8.73% | Plan a human review of visible, stale candidates; inspect metadata and SERP fit. |
| 3 | MONITOR | 2,875 | 1.63% | Track visibility and clicks; escalate only when evidence or business context changes. |
| 4 | DEFER | 145,274 | 82.20% | Leave out of the current review sprint and re-score later; this is not a prune recommendation. |

The queue may prioritize attention only. Publishing, deletion, pruning, merging, redirects, and causal claims require human judgment and are prohibited as automated outcomes.

## 7. Reproducibility

The repository contains all weekly notebooks, the complete capstone notebook, and machine-readable receipts for the model, validation audit, action playbook, and final summary.

- [Repository](https://github.com/Cooper30/flyrank-ml-internship)
- [Capstone notebook](https://github.com/Cooper30/flyrank-ml-internship/blob/main/work/notebooks/capstone.ipynb)
- [Run the capstone in Colab](https://colab.research.google.com/github/Cooper30/flyrank-ml-internship/blob/main/work/notebooks/capstone.ipynb?flush_cache=true)
- [Committed outputs](https://github.com/Cooper30/flyrank-ml-internship/tree/main/work/outputs)

## 8. Acknowledgments & data credit

[**Built on the FlyRank ML Internship dataset**](https://flyrank.ai). Thanks to FlyRank for providing the gated, real-world search dataset and the research workflow used for this capstone.

All public results are aggregated and public-safe. No client names, domains, URLs, private queries, credentials, or raw exports are published.
