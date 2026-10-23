# MoonTech Client Intelligence 2.0 — Source-Gap Register (Phase 0)

Every mockup feature that the **live data cannot truthfully support**, with: the exact field(s) required, who
would own them, and the **accurate interim alternative** V2 ships instead (never a fabricated number or chart).

---

## A. Blocked features → required fields → interim alternative

| # | Mockup feature (pages) | Why blocked | Required new field(s) | Proposed owner / system | Interim alternative (ships now) |
|---|---|---|---|---|---|
| G1 | Acquisition **source/channel** donut — Paid Social / Outbound / Referral / Partner / Organic (P1, P3) | No channel/source field; invoice `Brand="New Brand"` is not a channel | `channel`, `source`, `campaign` per acquisition event, keyed to brand | Growth/CRM (HubSpot) → `Acquisition_Attribution` sheet | "New brands by paid-M1 entry month" or ROAS-vs-non-ROAS split, labelled by basis |
| G2 | **CAC** "$980 per new brand" (P3) | No attributable acquisition/marketing cost | `acquisition_cost`, attribution method | Marketing finance | Rename to "median first paid invoice amount"; no CAC shown |
| G3 | **Campaign ROAS multiple** "4.1×", ROAS column, "Average ROAS" (P1, P4) | No ad spend or attributed GMV | `ad_spend`, `attributed_sales`, `currency`, `period` per campaign | Ad platforms / `Campaign_ROAS` sheet | Remove ROAS-multiple; show paid-milestone net **invoice spend** + spend momentum, clearly labelled |
| G4 | **ROI 6.1× / payback 2.4 months / "allocate $284K spend"** (P2) | No promotion budget, margin or payback economics | `planned_spend`, `margin`, `payback_def`, `approval` | Commercial / `Commercial_Plan` sheet | ARPA-uplift only (modelled $), no investment/ROI labels |
| G5 | **5-stage onboarding funnel** Paid→Setup→Data Access→QA→Live with per-stage % (P6) | Tracker has snapshot statuses, not dated stage events | `stage`, `entered_at`, `exited_at` per brand per stage | Ops / `Onboarding_Events` sheet | `Dashboard Integration` **status distribution** + duration histogram |
| G6 | **Owner** column + **SLA risk** (On track/At risk/Overdue) (P6) | No owner or SLA definition in source | `owner`, `sla_target_days`, per-stage SLA | Ops | Omit owner; optional read-only "at-risk" from transparent duration threshold (labelled, not "SLA") |
| G7 | **Top blockers / named reasons** ("No data access", "Missing permissions"…) (P6) | No blocker-reason field | `blocker_code`, `blocker_note` | Ops `Onboarding_Events` | Observable issues: unmatched brand, not-completed, on-hold, no first-ad after paid M1, negative/invalid days, missing dates |
| G8 | **Onboarding timeline** w/ Setup/QA dates + "Send reminder" button (P6) | Only M1-paid, first-ad (and maybe integration) dates exist; no reminder system | stage timestamps; task/messaging integration | Ops + tasking tool | Show only real-dated events; others "date unavailable"; no fake reminder CTA |
| G9 | **Historical churn / retention curves** by cohort, "December retention dip" (P3, P5) | Snapshot `Status` only; `Churn Date` gives timing for currently-churned brands but not full lifecycle history | dated `status`, `effective_date`, `actor` events (monthly snapshots) | Ops / `Lifecycle_Events` sheet | Paid-milestone **attainment** proxy (censored cohorts) + churn-by-month from `Churn Date` (labelled limited) |
| G10 | **Payment history timeline** — reminder sent / email / phone follow-ups (P7) | Only invoice-issue + one payment-receipt date normalized | dated reminder/contact events | A/R workflow | Show the two real events (issued, received) only |
| G11 | **Lead → paid conversion** funnel / the manual `75` leads KPI (P1) | No CRM lead records; `75` is a manual assumption | lead-level records with dates | HubSpot | Demote to explicit **what-if** control (not a KPI) |
| G12 | **Projected ARPA by month** trajectory (P2) | No scheduled/dated promotion plan | dated promotion schedule per brand | Commercial | Single current-vs-after bar; monthly trajectory only if a dated plan is supplied |
| G13 | **Download PDF** (P7) | Not implemented | — | — | Omit or disable; CSV/XLSX export ships |
| G14 | True **historical A/R snapshots** (point-in-time status) | Today's `Paid` status hides what was unpaid in the past | status/payment event history | A/R | Current as-of A/R only; mark historical A/R unsupported |

## B. Partial enablers found in live data (upgrade opportunities, not blockers)
- **Payment Method** (Feed A) — enables cash-vs-bank reconciliation / method mix (not in mockups; optional).
- **Churn Date** (Feed B) — limited churn-by-month timing for currently-churned brands.
- **Date of First Payment, M1/M2 Completion Date, Current Milestone** (Feed B) — tracker-side milestone timing to corroborate invoice-derived velocity.
- **HubSpot Company ID** (tracker's "Milestones for Data Team" tab) — the **join key** that would unblock G1/G2/G11 if HubSpot is connected.
- **Retain-now (already present, just unmapped):** Payment Terms, Days-between-invoicing-and-payment, Amendment Details.

## C. Proposed supplemental sheets (governed, only if business supplies them)
Define owner, schema version, canonical join ID (BrandID / HubSpot Company ID), freshness and governance **before** wiring any gated chart.

| Sheet | Columns | Unblocks |
|---|---|---|
| `Acquisition_Attribution` | BrandID, firstPaidEventID, channel, campaign, source_cost, owner, date | G1, G2, G11 |
| `Lifecycle_Events` | BrandID, status, effective_date, actor | G9, G14 |
| `Onboarding_Events` | BrandID, stage, entered_at, exited_at, owner, blocker_code, SLA_target | G5, G6, G7, G8 |
| `Campaign_ROAS` | BrandID, campaignID, period, currency, ad_spend, attributed_sales, source | G3 |
| `Commercial_Plan` | BrandID, planned_spend, approval, margin, date | G4, G12 |

**Rule:** gated modules keep their shell with a quiet "Data integration required — needs `<field>`" state (never a
persuasive fake chart), and are replaced by the listed interim P1 visual until the real source exists.
