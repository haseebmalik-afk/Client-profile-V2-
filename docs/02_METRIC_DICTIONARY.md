# MoonTech Client Intelligence 2.0 — Metric Dictionary / Registry (Phase 0)

One canonical definition per metric. Every chart/KPI/table in V2 calls these shared selectors — no component
recomputes a metric its own way. `date_basis` is explicit everywhere. Currency: AED is gross invoice money;
USD is `$ Amount`-derived net (never double-converted on display toggle). Status: **Actual** unless marked **Scenario**.
Tier P0 = already supported · P1 = safe derived (validate + show n) · P2 = blocked (see gap register).

Legend for `date_basis`: **inv** = Invoice Date · **pay** = Payment receipt date · **asof** = sheet as-of (10/22/2026) · **tracker** = tracker-supplied date.

---

## P0 — Baseline actuals (exist in current data; must reconcile to V1 exactly)

| metric_id | label | formula (numerator / denominator) | amount_basis | date_basis | currency | exclusions | source lineage | actual/scenario |
|---|---|---|---|---|---|---|---|---|
| `cash_collected` | Cash collected | Σ `Total` where Status=Paid and payDate ∈ window | gross | pay | AED→disp | cancelled, unpaid | FeedA: Total, Invoice Status, Payment receipt date | actual |
| `paid_receipt_count` | Paid receipts | count distinct Paid invoices with payDate ∈ window | — | pay | — | — | FeedA Invoice | actual |
| `invoiced_value` | Invoiced in period | Σ `Total` where not cancelled and invDate ∈ window | gross | inv | AED→disp | cancelled | FeedA Total, Status, Invoice Date | actual |
| `paid_billed_value` | Billed-and-paid (period) | Σ `Total` Paid and invDate ∈ window | gross | inv | AED→disp | ≠ cash_collected | FeedA | actual |
| `open_billed_value` | Billed-and-open (period) | Σ `Total` not-paid not-cancelled, invDate ∈ window | gross | inv | AED→disp | paid, cancelled | FeedA | actual |
| `open_receivables_asof` | Open A/R (as-of) | Σ `Total` not-paid not-cancelled outstanding ≤ as-of (all history) | gross | asof | AED→disp | paid, cancelled | FeedA | actual |
| `overdue_receivables` / `_count` | Overdue A/R + count | open invoices with Due < as-of (validated) | gross | asof | AED→disp | not-yet-due | FeedA Due date, Status | actual |
| `cancelled_value` / `_count` | Cancelled (period) | Σ / count cancelled, invDate ∈ window | gross | inv | AED→disp | — | FeedA | actual |
| `arpa_model_annualised_usd` | Modelled annualised ARPA | Σ per-brand modelled USD ÷ qualifying paid-ROAS brand count | **net USD** | pay/asof | USD | non-ROAS paid (unless in `ARPA_INCLUDE_BRANDS`) | FeedA $ Amount/Total/VAT/Refunds/Milestone/Status + FeedB churn | **actual (modelled)** |
| `arpa_participant_count` | ARPA brands (n) | count of qualifying paid-ROAS brands | — | — | — | — | FeedA/FeedB | actual |
| `excluded_paid_nonroas_count` | Excluded paid non-ROAS | count paid brands with no paid milestone & not in include list | — | — | — | — | FeedA | actual |
| `usd_net_invoice` | Net USD per invoice | `$ Amount` else `Total/3.6725`; ÷1.05 if VAT Included; − refund; floor 0 | net | — | USD | — | FeedA $ Amount, VAT, Refunds | actual |
| `highest_paid_roas_milestone` | Paid M level | max(milestone) over Paid rows per brand | — | pay | — | pending milestones | FeedA | actual |
| `active_paying_roas_brands` | Active paying ROAS brands | count brands with ≥1 paid ROAS milestone, not churned/dropped | — | asof | — | churned, dropped | FeedA + FeedB status | actual |
| `cash_confirmed_new_acquisitions` | New brands won | brands with Brand="New Brand" dated by first **paid** receipt + approved `REACQUIRED` | — | pay | — | unpaid new brands | FeedA Brand, Payment receipt date; CONFIG.REACQUIRED | actual |
| `roas_total_net_paid_usd` | ROAS net paid (lifetime/period) | Σ `usd_net_invoice` over paid ROAS invoices | net | pay | USD | — | FeedA | actual |
| `avg_days_to_integrate` | Avg days to integrate | mean valid `Days for Integration` (tracker) | — | tracker | — | negatives, blanks | FeedB Days for Integration | actual |
| `avg_days_payment_to_first_ad` | Payment→first ad | First Ad Live Date − paid-M1 payDate; report completed vs waiting | — | tracker/pay | — | missing/negative flagged | FeedB First Ad + FeedA M1 payDate | actual |
| `tracker_status_count` | Tracker status mix | count by Status (Active/Churned/On hold) | — | asof | — | Average row | FeedB Status | actual |
| `source_match_rate` | Join coverage | invoice brands matched in tracker ÷ eligible invoice brands | — | — | — | — | FeedA brand ∩ FeedB brand (+overrides) | actual |

## P1 — Safe derived (implement after validation; always show n / window)

| metric_id | label | formula | basis | date_basis | notes |
|---|---|---|---|---|---|
| `avg_payment_delay` | Median/mean payment delay | median & mean (payDate − invDate) for Paid rows, both dates valid | days | inv→pay | exclude negative/corrupt; expose n |
| `collection_punctuality` | On-time collection | paid on/before Due ÷ paid with valid Due | ratio | due/pay | not the same as collection rate |
| `receivables_ageing` | Ageing buckets | open `Total` in Not-due / 0–30 / 31–60 / 61–90 / 91+ at as-of | gross | asof | buckets from Due vs as-of |
| `revenue_concentration` | Top-5 / Top-10 share | paid cash of top brands ÷ all paid cash in window | ratio | pay | invoice-receipt proxy; warn if denom 0 |
| `avg_cash_per_new_brand` | Cash per new cohort brand | cash from new-brand cohort ÷ cohort size (fixed window) | AED | pay | label cohort; don't divide all cash by new brands |
| `new_to_m2_m3_conversion` | M1→M2 / M1→M3 attainment | brands with paid M1 ever reaching paid M2/M3 ÷ eligible M1 cohort | ratio | pay | require observation window; censor immature |
| `milestone_progression_time` | M1→M2, M2→M3 … days | median & P75 days between first paid dates of consecutive milestones | days | pay | distinct events; show n |
| `milestone_distribution` | Paid M1–M5 counts | count brands by highest paid milestone (+ net spend) | — | pay | not tracker tier; not ad ROAS |
| `revenue_trend_monthly` | Monthly cash receipts | Σ paid `Total` by payDate month (12–18 mo) | gross | pay | current month marked partial |
| `invoice_status_mix` | Status by count/amount | count (and optional amount) by normalized status | — | inv | one basis per chart |
| `tracker_match_coverage` | Matched / unmatched lists | matched, ambiguous, unmatched brand sets | — | — | surface specific exceptions |
| `time_to_integrate_dist` | Integration duration | mean/median/P75 + histogram of valid `Days for Integration` | days | tracker | flag negatives |
| `promotable_book` | M1/M2 under target | non-dropped M1/M2 brands below $24K, ranked by modelled uplift | USD | — | precise denominator |
| `scenario_arpa_uplift` | ARPA uplift (what-if) | projectedARPA(n1,n2) − current ARPA; %=uplift/current (if ≠0) | net USD | — | **SCENARIO**; $36K locked M3 from config |
| `acquisitions_by_month` | Monthly acquisitions | count cash-confirmed new brands by month; reacquisitions flagged separately | — | pay | — |
| `ab_brand_contribution` | A vs B brand delta | paid cash by brand in A & B; added/removed/continuing; reconciles to aggregate Δ | gross | pay | descriptive, not causal |
| `churn_timing` (NEW, provisional) | Churn by month | count churned brands by `Churn Date` month | — | tracker | **limited**: snapshot date only, not full lifecycle; label clearly |

## Reconciliation gate (acceptance)
For an identical snapshot, every P0 metric must equal the V1 engine **to unrounded precision**. Specific cases
to lock: VAT-included netting (`1000/1.05`), refund subtraction + floor-0, zero `$ Amount` fallback, Jan-invoice/
Feb-payment split, pending milestone stays at paid level, dropped M3+ switches to YTD, churn auto-drop ∪ manual
drop, A=0 comparison shows "New"/n-a (never +100%), AED↔USD toggle never double-converts ARPA. Deliverable:
old-vs-new CSV with **zero** unexplained variance on unchanged metrics.
