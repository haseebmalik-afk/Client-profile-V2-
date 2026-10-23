# MoonTech Client Intelligence 2.0 — Seven-Page Implementation Plan (Phase 0)

Navigation keeps the original seven reports in order. Each page: shell → header + period/currency controls →
4–5 KPIs → 1–2 primary visuals → secondary cards → drill-down table. Status per module:
**🟢 IMPLEMENT NOW** (supported) · **🟡 DERIVE & VALIDATE** (safe derivation, show n) · **🔴 BLOCKED** (needs
new source; render a truthful alternative + gap item, never fake numbers). Mockup sample figures (ARPA $6,420,
$284,600 overdue, 112 brands, CAC $980, ROAS 4.1×, owner names, SLA, +35% badges) are **layout only** — all values recomputed from source.

---

## Page 1 — Overview & ARPA
**Layout:** 5 KPIs → ARPA audit trail + Collections ageing → Path-to-target + compact ladder + ARPA teaser → Monthly trend + Acquisition log.

| Module | Status | Source / formula |
|---|---|---|
| 5 KPIs: Cash collected, Modelled ARPA, Active paying ROAS brands, Open receivables, New brands won | 🟢 | `cash_collected`, `arpa_model_annualised_usd`, `active_paying_roas_brands`, `open_receivables_asof`, `cash_confirmed_new_acquisitions` |
| Total invoiced / paid / cancelled | 🟢 (demote to billing drawer) | P0 billing metrics — not hero cards |
| ARPA workings audit trail (per-brand, collapsible) | 🟢 | `computeARPA` rows: net USD, basis, dropped, exceptions, count |
| Collections & ageing bars (Not-due/0-30/31-60/61-90/91+) | 🟡 | `receivables_ageing` (validate Due vs as-of) |
| Compact ARPA upgrade teaser (current vs projected) | 🟢 (scenario-labelled) | `scenario_arpa_uplift`, never overwrites actual KPI |
| Path to $24K table (top eligible, gap bar) | 🟡 | `promotable_book` |
| ROAS ladder summary counts (link to Velocity) | 🟢 | `milestone_distribution` |
| Monthly trend: cash bars + acquisitions line, partial month hatched | 🟢 | `revenue_trend_monthly` + `acquisitions_by_month` (dual-axis, labelled) |
| Acquisition log (grouped by month) | 🟢 | FeedA new-brand rows |
| Collection punctuality / median payment days / top-overdue | 🟡 | `collection_punctuality`, `avg_payment_delay` |
| "ARPA build to $10K" (mockup target) | 🔴→fix | Use CONFIG **$24K**, not mockup $10K |

## Page 2 — Brand Build
**Layout:** 5 stats → ARPA uplift planner (2 sliders) + projected-uplift panel + current/plan/target bars → Path-to-$24K priority table + tier funnel → opportunity matrix.

| Module | Status | Source / formula |
|---|---|---|
| 5 stats: Current ARPA, Projected ARPA, Brands below target, Total modelled uplift, Eligible promotions | 🟢 | live ARPA + `promotable_book` |
| ARPA uplift planner (M1→M3, M2→M3 sliders, brand lists) | 🟢 | `scenario_arpa_uplift`; 0 promos = baseline exactly |
| Current / Plan / Target 3-bar chart | 🟢 | current actual, scenario, CONFIG $24K |
| Path to $24K priority table (gap, value after M3=$36K, tier) | 🟡 | `promotable_book`, deterministic ranking |
| Brand tier funnel by **paid M-level** counts | 🟡 | `milestone_distribution` (NOT tracker Dragon/Eagle tiers; not a sales funnel) |
| Opportunity matrix (level × uplift band) | 🟡 | modelled uplift only; axes never labelled "$ investment/spend" |
| **Projected ARPA by month** trajectory | 🔴 | No scheduled promotion dates → use single current-vs-after bar unless user supplies a dated promotion plan |
| **Recommended actions** (heuristic, which brands + why) | 🟡 | deterministic (highest uplift / closest to M3) — no AI causal stories |
| **"Allocate $284K spend, 6.1× ROI, 2.4-mo payback"** | 🔴 | No cost/margin/payback source → omit (gap item) |

## Page 3 — Growth
**Layout:** 5 cards → Monthly acquisitions + Cash-vs-brands charts → cohort quality + acquisition-basis + insights → acquisition log.

| Module | Status | Source / formula |
|---|---|---|
| Cards: New brands won, Collected revenue, Cash from new-brand cohort, Median first-month spend, Acquisition momentum | 🟢/🟡 | `cash_confirmed_new_acquisitions`, `cash_collected`, `avg_cash_per_new_brand`, median first paid invoice, MoM growth |
| Monthly brand acquisitions (bars + cumulative line, partial month) | 🟢 | `acquisitions_by_month` |
| Collected revenue vs brands won (dual-axis, labelled) | 🟢 | `revenue_trend_monthly` + acquisitions |
| New-brand quality cohort heatmap | 🟡 | `new_to_m2_m3_conversion` as **paid-milestone attainment** proxy; censor immature cohorts; label as payment-activity (not true retention) |
| **Acquisition sources donut** (Paid Social/Outbound/Referral…) | 🔴 | No channel field. Replace with "New brands by paid-M1 entry month" or ROAS vs non-ROAS split |
| Growth insights (best month, MoM growth, % new reaching M2, new w/ open invoice) | 🟡 | deterministic, each cites source rows |
| Monthly acquisition log | 🟢 | FeedA + reacquisition flag + tracker status (no "campaign source") |
| **Avg first-month spend labelled "CAC $980"** | 🔴 | First-month spend ≠ CAC — rename to "median first paid invoice", no CAC |
| **"December retention dip −8pp"** retention curve | 🔴 | No dated lifecycle history → proxy only, labelled |

## Page 4 — ROAS Velocity
**Layout:** 5 cards → Milestone distribution + velocity trend → spend-growth-health bubble + tier transitions + health donut → movers/laggards + activity log.

| Module | Status | Source / formula |
|---|---|---|
| Cards: Active ROAS brands, Avg monthly net spend, Fastest-growing (>50% spend growth), Pending invoices, Ladder health | 🟢/🟡 | counts + `roas_total_net_paid_usd` MoM + health rule-based |
| Paid milestone distribution M1–M5 (count + net spend) | 🟡 | `milestone_distribution` |
| Velocity trend (cumulative reaching paid M1/M2/M3; or monthly net spend) | 🟡 | `milestone_progression_time` series on paid dates |
| Spend vs growth vs health bubble | 🟡 | X=net ROAS spend, Y=% MoM spend change, color=rule-based health. Labelled **invoice-spend momentum, not ad ROAS** |
| Tier transitions (Sankey) | 🟡 | only **paid-milestone** A→B transitions reconstructed from first-paid dates; not tracker-tier changes |
| Ladder health donut | 🟡 | rule-based status counts + caveat |
| Top movers / laggards | 🟡 | ranked by % change in paid net ROAS spend / new paid milestone; laggard reasons from source facts (no invoice / cancelled / no first ad / dropped) |
| Recent ladder activity log | 🟢 | paid milestone events only (no fabricated "upgraded") |
| Time to first ad | 🟢 | `avg_days_payment_to_first_ad`, completed vs waiting, flag negatives |
| **"Average ROAS 4.1×" / "ROAS" column** | 🔴 | No ad-spend/GMV feed → remove ROAS-multiple column; M1–M5 are payment stages |

## Page 5 — Compare Periods
**Layout:** A/B period pickers (swap, equal-length helper) → 4–5 paired KPI cards → grouped bars + two-period trend → revenue bridge + insights → brand-level table.

| Module | Status | Source / formula |
|---|---|---|
| Paired KPIs: Brands won, Cash collected, Modelled ARPA, (median days to pay), Active paid brands — A, B, Δ, % (if A≠0), period length | 🟢 | P0 metrics per period |
| Grouped bars (separate $ and # subcharts) | 🟢 | no mixed y-axis |
| Two-period trend (day-aligned or calendar) | 🟢 | `revenue_trend_monthly` windowed |
| Revenue bridge/waterfall (Start A + new/increase − lost/decrease = End B) | 🟡 | `ab_brand_contribution`; must reconcile within tolerance; **no causal labels** |
| Key insights (3–5, deterministic, drill-down) | 🟡 | from computed deltas only |
| Brand-level comparison table (A/B cash, ARPA, Δ, paid M level, status) | 🟡 | `ab_brand_contribution`; Added/Removed/Continuing (not "churned") |
| A=0 % handling | 🟢 | show "New"/n-a, never +100% |

## Page 6 — Integration
**Layout:** 5 cards → status distribution + time-to-integrate trend/histogram → integration×tier + first-ad delay → observable-issues list → tracker table.

| Module | Status | Source / formula |
|---|---|---|
| Cards: Brands in onboarding, Avg time to integrate, (Overdue setups*), Go-live/first-ad rate, (Blocked*) | 🟢/🟡 | tracker status counts, `avg_days_to_integrate`, first-ad coverage |
| **5-stage pipeline funnel** (Paid→Setup→Data Access→QA→Live) | 🔴 | No dated stage events → replace with **Dashboard Integration status distribution** |
| Integration status × Client Tier (stacked bars) | 🟡 | FeedB integration × tier |
| Time-to-integrate: histogram + median/P75 (trend only if completion date valid) | 🟡 | `time_to_integrate_dist`; no false monthly trend from snapshots |
| Time to first ad (completed vs waiting, anomalies) | 🟢 | `avg_days_payment_to_first_ad` |
| **Top blockers / issues** (named reasons) | 🔴 | No blocker-reason field → show observable issues: unmatched brand, not completed, on hold, no first-ad after paid M1, invalid/negative days, missing dates |
| **Onboarding timeline w/ Setup/QA dates + "Send reminder"** | 🔴 | Only M1-paid / first-ad / (maybe integration) dates real; others "date unavailable"; no fake reminder action |
| **Owner column + SLA risk** | 🔴 | No owner/SLA source → omit; optional read-only "at-risk" via transparent duration threshold (not "SLA") |
| Tracker table (brand, tier, status, integration, days, first-ad, payment→first-ad, current vs paid milestone, match status) | 🟢 | FeedB + join; search/filter/export |

## Page 7 — Client Ledger
**Layout:** 5 cards → ageing bars + status donut + top-outstanding → ledger table (hero) with expandable rows.

| Module | Status | Source / formula |
|---|---|---|
| Cards: Total invoices, Collected (cash period), Outstanding (as-of), Overdue (as-of), Active clients — each labelled inv-date vs pay-date | 🟢 | P0 metrics |
| Ageing bars (Not-due/0-30/31-60/61-90/91+, count + AED) | 🟡 | `receivables_ageing`, gross balances until partial-payment data exists |
| Invoice status donut (by count; amount toggle) | 🟡 | `invoice_status_mix`, one basis per chart |
| Top outstanding brands (ranked, % of total, click-to-filter) | 🟡 | open balance by brand |
| Ledger table (hero): sticky invoice/brand, issue/due/paid dates, net/gross, M stage, status, age, currency, refunds | 🟢 | FeedA full rows, source IDs preserved |
| Expandable row (all source fields incl. retained Payment Terms, Amendment, Payment Method) | 🟢 | FeedA; exact, no fabrication |
| **Payment history timeline** (reminder sent / followed up email / phone) | 🔴 | Only invoice-issue + one payment-receipt date exist → show those two events only |
| Export CSV/XLSX w/ provenance footnotes | 🟢 | existing XLSX + lineage |
| **"Download PDF"** | 🔴 | Not implemented → omit or clearly disable |

---

## Build order (engine-first; nav order unchanged)
Phase 1 shared data layer + metric registry + source-state banners → Phase 2 tokens/shell/charts + **Overview & ARPA
in both themes for approval** → Phase 3: Overview → Brand Build → Client Ledger → Growth → ROAS Velocity →
Compare → Integration → Phase 5 regression + 14 screenshots (7 light/7 dark) + mobile + old-vs-new reconciliation CSV.
All on a branch in `Client-profile-V2-`; nothing to production without approval.
