# MoonTech Client Intelligence 2.0

A seven-page finance & client-intelligence dashboard for MoonTech's ROAS business, rebuilt from the V1
"Client Profile" dashboard. Single-file static app (`index.html`), deployed via GitHub Pages, reading two
published Google Sheets (receivables + ROAS tracker) client-side via PapaParse.

**This is a V2 test build in a separate repository.** It preserves every financial rule from V1
(invoice-vs-payment date semantics, VAT netting, refunds, AED/USD peg, paid-milestone ARPA model,
approved exceptions, churn/drop logic, $24K target, $36K M3 value, XLSX export) and rebuilds the
presentation layer to match the seven approved mockups, in polished light **and** dark modes.

## Reports
1. **Overview & ARPA** — 5 KPIs, ARPA audit trail, collections ageing, path-to-target, ladder, monthly trend, acquisition log
2. **Brand Build** — ARPA uplift planner (M1/M2 → M3 sliders), projected uplift, priority table, tier funnel, opportunity matrix, deterministic recommendations
3. **Growth** — acquisitions, cash-vs-brands, cohort attainment proxy, acquisition-basis (gated channel alternative), insights, log
4. **ROAS Velocity** — milestone distribution, velocity trend, spend/growth/health bubble, timing, health, movers/laggards
5. **Compare Periods** — A/B KPIs, grouped bars, reconciled revenue bridge, brand-level delta table
6. **Integration** — status distribution (not a fake funnel), time-to-integrate, integration×tier, observable issues, tracker table
7. **Client Ledger** — ageing, status donut, top outstanding, expandable invoice ledger

## Honest data governance
Mockup placeholders that the live data cannot support (acquisition channel/CAC, campaign ROAS multiples,
ROI/payback, 5-stage onboarding funnel, owner/SLA/blocker, retention curves, reminder timelines, PDF export)
are **gated** with a "data integration required" note and replaced by a truthful alternative — never fabricated
numbers. See `docs/` for the full source-data audit, metric dictionary, seven-page plan and source-gap register.

## Data sources (published CSV, set in `CONFIG`)
- Receivables: *Receivables Status Sheet 3.0* → "Moon Tech – Entity" tab
- Tracker: *ROAS Tracker Sheet* → "Tracker" tab

Source state (Live / Sample / Unavailable) is shown explicitly; the app never passes sample data off as live.
