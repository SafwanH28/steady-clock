# Steady Clock, lock 2026Q3: pre-registration

Written before any estimate in this lock was sealed. The SHA-256 of this file is the first leaf of the lock's Merkle root. Nothing in it changes after the seal. Any analysis not described here will be labelled exploratory.

## 1. What is forecast

For each company in the universe, the fiscal quarter whose three-month period ends between 2026-09-10 and 2026-10-10:
- **revenue**, in USD;
- **diluted EPS**, in USD per share.

## 2. Universe

A company is in the universe if all of the following hold:
- its common stock trades on Nasdaq or NYSE;
- it is a domestic SEC filer, not a foreign private issuer;
- it traded on or after 2026-10-01;
- first-filed revenue for its quarter ending between 2026-06-10 and 2026-07-10 was at least $1M and was filed by 2026-10-09;
- revenue is on file for its quarter ending between 2025-09-10 and 2025-10-10;
- its SIC code is not 6770 (blank checks).

The source is SEC XBRL (companyfacts), loaded 2026-10-10. `universe.json` lists every company; its hash is a leaf.

## 3. Inputs

Each company's packet holds only SEC XBRL values filed on or before **2026-10-09**, taking the first-filed copy of each period:
- three-month quarters since mid-2023: revenue, gross profit, operating income, net income, diluted EPS (basic where diluted is not tagged), diluted shares;
- fiscal-year totals since mid-2022;
- the latest cash and long-term debt.

The packets contain nothing else: no news, prices, analyst data or web access. Each packet is hashed, and the Merkle root of the packet hashes is a leaf.

## 4. Arms

| Arm | Definition |
|---|---|
| **ai:gpt-6-luna** (primary) | Every non-void company. One call per company, low reasoning effort, strict JSON output: revenue and EPS at p10, p50 and p90. The prompt hash is a leaf. |
| **ai:gpt-6.1-sol** (secondary) | A size-stratified sample (every k-th company by last-quarter revenue), same prompt |
| **B1** | Seasonal random walk: the target equals the same quarter a year earlier |
| **B2** | Revenue = the target quarter a year earlier × (last quarter / the same quarter a year before it). EPS = the target quarter a year earlier + (last quarter's EPS − the same quarter a year before it). Missing inputs fall back to B1. |

Failed AI calls (invalid or no answer) are sealed as failures and counted. In the primary test, a failed luna row is scored with the B1 value, which is the fallback the app would show.

## 5. Void rule

A company is void if, after 2026-10-09 and before its row's `issued_at`, it filed either of these with the SEC:
- a 10-Q or 10-K whose report date falls in the target window;
- an 8-K with Item 2.02.

This is checked at the lock against the EDGAR daily indexes and the company's submissions (`void.json`, hashed as a leaf), and again at grading against EDGAR acceptance times. A company with an 8-K under Item 7.01 or 8.01 in that window is flagged as a possible pre-announcement and reported separately; it stays in the test.

## 6. Truth

**Revenue:** the first-filed three-month value (80 to 100 days) for the target period. Concepts in order of preference:
1. Revenues
2. RevenueFromContractWithCustomerExcludingAssessedTax
3. RevenueFromContractWithCustomerIncludingAssessedTax
4. SalesRevenueNet

**EPS:** the first-filed three-month EarningsPerShareDiluted, falling back to EarningsPerShareBasic.

Other rules:
- **Reverse splits:** if the company splits between the cutoff and the filing, EPS estimates are converted by the split ratio before scoring.
- **Deadline:** values not filed by 2026-11-30 are unresolved. They are excluded from scoring, but counted and reported.
- **Review:** any absolute error above 50% of the actual goes to manual review for tag, scale or period mistakes. Every correction is logged as a new record.

## 7. Primary test

1. **Scale.** For each company, the scale s is the mean of |revenue(t) − revenue(t−4)| over the quarters in its packet where both values exist, up to the latest 8. Companies with fewer than 2 such pairs, or s = 0, are excluded from scaled metrics and counted.
2. **Statistic.** R = mean(|luna p50 − actual| / s) ÷ min( mean(|B1 − actual| / s), mean(|B2 − actual| / s) ), taken over companies graded for luna, B1 and B2.
3. **Pass.** R < 0.95, **and** the upper end of its 95% interval is below 1.00. The interval comes from a bootstrap that resamples whole 2-digit SIC industries (2,000 draws, seed 20261024).

## 8. Secondary analyses (Holm-corrected at a family-wise 5%, reported whatever they show)

- **a.** The EPS MASE ratio, with EPS scale defined the same way. Companies whose actual |EPS| is below $0.05 are excluded.
- **b.** luna p10 to p90 revenue interval coverage (target 80%), with a binomial interval.
- **c.** Pinball loss for luna's three quantiles.
- **d.** sol against luna on the sol sample, as a MASE ratio.
- **e.** The primary statistic by size tier (tertiles of last-quarter revenue).
- **f.** Flagged possible pre-announcers, reported separately.
- **g.** Rows flagged `scale_check` (a p50 outside 0.2x to 5x of the last quarter), reported separately.

## 9. Publication

- **At the lock:** the manifest (counts, root, meta hashes) and this file.
- **After grading (about 2026-11-30):** every row with its Merkle proof, the actuals, and the results, whatever they show.
- **Wording:**
  - "AI estimate", never "consensus".
  - No buy or sell language.
  - No estimate shown while its quarter is live.
  - Accuracy claims made only from this test.
