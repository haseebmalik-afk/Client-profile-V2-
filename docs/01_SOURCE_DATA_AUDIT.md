# MoonTech Client Intelligence 2.0 — Source-Data Audit (Phase 0)

**Prepared for approval before any code.** Audit date: 22 Oct 2026. As-of date carried by both feeds: **10/22/2026**.
No raw client rows are reproduced here or committed to the repo; only schema, structure and population characteristics.

---

## 1. Feeds identified (ground truth)

The dashboard's `CONFIG` embeds two published-to-web CSV feeds. By matching the code's required column
aliases (`HKEYS` / `TKEYS`) against the live Google Drive source cells, both feeds are identified with high
confidence. One confirmation is requested (see §6).

| Code constant | Role | Source workbook (Drive) | Source tab | gid | Rows (approx.) | Currency |
|---|---|---|---|---|---|---|
| `SHEET_CSV_URL` | Receivables / ROAS invoices (**Feed A**) | **Receivables Status Sheet 3.0** (owner: haseeb.malik, mod. 22 Oct 2026) | **"Moon Tech – Entity"** | 634629514 | ~76 invoices | AED gross + USD `$ Amount` |
| `TRACKER_CSV_URL` | ROAS onboarding tracker (**Feed B**) | **ROAS Tracker Sheet** (owner: nora.tawfik, mod. 22 Oct 2026) | **"Tracker"** | 793116860 | ~46 brands (+1 "Average" row stripped) | n/a (status/dates) |

Why these tabs specifically: "Moon Tech – Entity" is the **only** tab in the receivables workbook that carries
`Brand Name`, `Brand` (New-Brand flag), `ROAS- Milestone`, `VAT`, `$ Amount` and a refunds column together —
the exact set `HKEYS` resolves. The workbook's other tabs (`2020`–`2026`, `KSA- Entity`, `G suite`, the
`Jamjoom series …` tabs) are the **broader multi-entity A/R ledger** (all ITCAN invoicing, AED/SAR, no
Brand/ROAS/USD columns) and are **not** the ROAS dashboard feed. Likewise the tracker workbook holds many
per-brand milestone tabs (`Goldenscent`, `Rightbite`, …) and a richer `Milestones for Data Team` tab; the
dashboard consumes only the flat `Tracker` tab.

---

## 2. Feed A — "Moon Tech – Entity" schema (receivables)

Exact live header order (cols A→U):

`Invoice · Client Name · Brand Name · Description · Brand · ROAS- Milestone · Invoice Date · Payment Terms · Due date · Days past due · Total · Invoice Status · Payment Method · VAT · $ Amount · Refunds VAT Exclusive · Payment receipt date · Days between invoicing and payment · Todays Date · 10/22/2026 · Ammendment Details`

| Column | Dashboard `HKEYS` mapping | Status | Notes |
|---|---|---|---|
| Invoice | `invoice` | ✅ mapped | Row ID / drill-down / export |
| Client Name | `client` | ✅ mapped | Legal entity (e.g. "DREAMABLE - Moontech") |
| Brand Name | `brandCol` | ✅ mapped | Grouping key (e.g. "Smallable"); falls back to derived client name if blank |
| Description | `desc` | ✅ mapped | Invoice detail |
| Brand | `brand` (New-Brand flag) | ✅ mapped | Value "New Brand" marks a qualifying acquisition |
| ROAS- Milestone | `mile` | ✅ mapped | "Milestone-1"…"Milestone-5", "Stage 1", or "N/A"; parsed to 1–5 |
| Invoice Date | `invDate` | ✅ mapped | MM/DD/YYYY (US locale) |
| Payment Terms | `terms` | ⚠️ **mapped but dropped** by `normalizeRows` | Present & populated ("0", "30") — retain in V2 |
| Due date | `due` | ✅ mapped | Ageing basis |
| Days past due | `pastDue` | ✅ mapped | Mostly "N/A" in sample → parses to 0; **validate against Due vs as-of** |
| Total | `total` | ✅ mapped | Gross AED (some rows SAR in other tabs; this tab AED) |
| Invoice Status | `status` | ✅ mapped | Paid / Cancelled / Unconfirmed / Open |
| **Payment Method** | — **unmapped** | 🆕 new field | "Bank Transfer" etc. Not in `HKEYS`. Candidate new dimension (cash vs card reconciliation). |
| VAT | `vat` | ✅ mapped | Values seen: "VAT Exclusive". Net division only triggers on "Included". |
| $ Amount | `usd` | ✅ mapped | Direct USD net basis (e.g. "$8,171.97"); primary ARPA input |
| Refunds VAT Exclusive | `refund` | ✅ mapped | Matches alias `refunds vat exclusive`; currency inferred from "$"/"AED" text |
| Payment receipt date | `payDate` | ✅ mapped | Cash-basis timing & acquisition dating |
| Days between invoicing and payment | `daysToPay` | ⚠️ **mapped but dropped** | Present & populated — retain in V2 (enables payment-delay metric) |
| Todays Date / 10/22/2026 | `today` | ✅ mapped | As-of reference = 10/22/2026 |
| Ammendment Details | `amend` | ⚠️ **mapped but dropped** | Retain in V2 (ledger expandable row) |

**Retain-in-V2 (zero new data, already present):** `Payment Terms`, `Days between invoicing and payment`,
`Ammendment Details`, `Payment Method`. Bringing these into the normalized row creates no fabricated data.

## 3. Feed B — "Tracker" schema (onboarding)

Exact live header order (cols A→AA): `Brand Name · Client Tier · Month of Signing · Priority · Status ·
First Payment Status · Client Intro Call Status · Dashboard Integration · Influencer Assignment ·
Current Milestone · Progress Achieved · Date of First Payment · Date of Client Intro · Date of Integration
(×2) · Days for Integration · Time from Payment to … · Brief and Coupons Received · Days from (Payment to …)
· time to campaign live · First Ad Live Date · Days from Payment to … · M1 Completion Date · Time to M1
completion · M2 Completion Date · Time to M2 completion · Churn Date`

Row 2 is an **"Average" summary row** — code correctly filters it (`!/^average$/i.test(brand)`).

| Column | `TKEYS` mapping | Status | Notes |
|---|---|---|---|
| Brand Name | `brand` | ✅ | Join key to Feed A brand |
| Client Tier | `tier` | ✅ | Dragon/Eagle/Kiwi-style — **distinct from M1–M5 milestones** |
| Status | `status` | ✅ | Active / Churned / On hold (string-matched) |
| Dashboard Integration | `integration` | ✅ | Snapshot status (not a dated stage event) |
| Current Milestone | `milestone` | ✅ | Tracker's own milestone (separate from paid-invoice milestone) |
| Days for Integration | `daysIntegrate` | ✅ | Integration report metric |
| Days from Payment to … (campaign) | `timeCampaign` | ✅ (alias match to verify) | Two similarly named cols exist — confirm which is "payment→campaign" |
| First Ad Live Date | `firstAd` | ✅ | Time-to-first-ad (minus paid-M1 date) |
| **Date of First Payment** | — unmapped | 🆕 | Tracker-side payment date; cross-check vs invoice payDate |
| **M1 / M2 Completion Date** | — unmapped | 🆕 | Supports milestone-progression timing from tracker side |
| **Churn Date** | — unmapped | 🆕 | Gives **churn timing** (month) for currently-churned brands — partial enabler for churn-by-month (still a snapshot, not full lifecycle history) |
| Integration Completion Date | `intDone` (in `TKEYS`) | ❌ **not present as named** | Only "Date of Integration …" cols exist; `ingestTracker` never retains it anyway. Confirms spec's warning — **no reliable completion-date time series**. |
| Priority, Progress Achieved, Influencer Assignment, Client Intro, Brief & Coupons, time to campaign live | — unmapped | optional | Available if a view needs them |

**Not present anywhere in Feed B:** onboarding **owner**, **SLA** rules/targets, **blocker reason** codes,
dated 5-stage pipeline events (Paid→Setup→Data Access→QA→Live), acquisition **channel/source**, **ad spend**,
**attributed revenue/GMV**, **lead** counts. These drive the mockup widgets that must be **BLOCKED** (see gap register).

**Bridge found:** the tracker's `Milestones for Data Team` tab carries a **HubSpot Company ID** per brand.
Not consumed today, but it is the natural join key if HubSpot is ever wired in to unblock channel/CAC/lead metrics.

---

## 4. Known data-quality risks (investigate, do not silently "fix")

1. **Sample fallback masquerading as live.** On CSV load failure the app silently renders an embedded sample dataset. V2 must show explicit **Live / Stale / Demo / Unavailable** source state; never pass sample as live.
2. **`Days past due` = "N/A"** on most rows → `parseAmt` coerces to 0. Overdue logic should be recomputed from `Due date` vs as-of, with stored value only used when validated.
3. **`parseAmt` → 0 for missing/malformed.** A true 0 is indistinguishable from "missing". Add a data-quality flag without changing legacy totals.
4. **`usdNet` fallback** `$ Amount || Total/3.6725`: a genuine $0 would fall back to AED. Test the zero case explicitly.
5. **Refund currency inferred from text** ("$"/"AED"). Column is "Refunds VAT Exclusive"; flag ambiguous rows rather than silently assuming USD.
6. **Date locale.** Slash dates parsed MM/DD/YYYY. Live sheet uses MM/DD/YYYY (e.g. 03/12/2025) — consistent, but flag any row that fails the pattern.
7. **Snapshot ≠ history.** Tracker `Status`/`Integration` are current-state only. No month-over-month status series exists (Churn Date gives limited timing only). Historical churn/retention curves remain unsupported.
8. **Currency toggle label drift.** Several V1 charts/tooltips/exports show "AED" even in USD mode — audit every label.
9. **Manual `leads = 75`** is an assumption, not a CRM figure — demote to a clearly-labelled what-if, never a KPI.
10. **Public exposure.** Both feeds are published-to-web (client-level receivables are world-readable via the CSV link). Flag for access-control review; keep raw rows out of the repo, logs and screenshots.

## 5. Preserved financial rules (carried unchanged into V2)
Invoice-date vs payment-date semantics · VAT netting only when "Included" · refund subtraction · AED/USD peg
3.6725 (display-only conversion; never double-convert USD-built ARPA) · paid-milestone level = max paid M ·
`computeARPA` basis rules (M1/M2 cumulative, M3+ latest×12, dropped M3+ → YTD) · `ARPA_INCLUDE_BRANDS`
(Jetour) · `BRAND_OVERRIDES` (elite cars→Jetour) · `REACQUIRED` (JustHype 2026-07) · churn auto-drop ∪ manual
drop · `ARPA_TARGET` $24,000 · `M3_MONTHLY` $3,000 → $36,000 annualised · XLSX export.

## 6. Confirmation requested
- **(C-1)** Confirm Feed A = *Receivables Status Sheet 3.0* → "Moon Tech – Entity", and Feed B = *ROAS Tracker Sheet* → "Tracker". (High confidence from column match; one-line yes/no.)
- **(C-2)** OK to pull the full row set of both tabs as the **first build step** to produce the exact unrounded old-vs-new baseline (no numbers are invented until then).
