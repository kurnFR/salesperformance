# Decision Log

Every open question has a **proposed default** so work can continue; the default applies only until the owner decides. Update status and date when a decision is made, and cite the DEC ID in PRs.

Status: `Confirmed` · `Proposed` (default in use, awaiting owner) · `Open` (blocked, no default) · `Superseded`.

## Confirmed

| ID | Decision | Confirmed | Where used |
|---|---|---|---|
| DEC-005 | Color rule: if achievement >= working-day progress -> green (even below 70%); otherwise <70% red, 70% to <100% yellow, >=100% green. Equality is green. Yellow overrides red when behind pace but >=70% | 2026-10-22 (owner, in chat) | `01-metrics-contract.md` section 4 |
| DEC-005a | Working-day progress is capped to 0-100% on the server | 2026-10-22 | `01-metrics-contract.md` 5.7 |
| DEC-005b | Working-day progress comes from the working-day table/calendar, not sales achievement | Earlier (owner) | `01-metrics-contract.md` 5.7 |
| DEC-005c | Role card example: RSM (5), IN 20% / 1 = one of five counted as achieved. The threshold behind "achieved" is now disputed: see DEC-021 | Earlier (owner) | `01-metrics-contract.md` 5.3 |
| DEC-020 | 70% up to 99% is yellow (70% inclusive) | 2026-10-22 (owner) | `01` RULE-009 |
| DEC-021 | Role cards use the **same rule as the bars**: an entity is "achieved" when achievement >= working-day progress (bar condition 1). The count is live: when working-day progress rises above an entity's achievement it stops counting (for example RSM IN goes from 1 to 0) | 2026-10-22 (owner) | `01` 5.3, `06` PERF-002 |
| DEC-022 | Principal is chosen in the web with buttons **ASW** and **INA**; both principals are in release 1. Access per principal is set by admin permissions, never hard-coded | 2026-10-22 (owner) | `06` DASH-004, `02` section 10 |
| DEC-011 | Data scope is per user and configurable: principal, region and other filters. Some users see one region, some several; only users granted full access see both principals. Roles and permissions are data managed by admins, not code | 2026-10-22 (owner) | `02` section 10, `03` section 3, `PRD` section 5 |
| DEC-024 | Active distributor % = `Total Aktif / TOTAL DIST` from table `distaktif` (sheet 2 region cards, for example 30 / 31 = 97%) | 2026-10-22 (owner) | `01` METRIC-016, `06` section 3 |
| DEC-027 | Working days come from the `dimdate` table (`networkeddays`); the app sums them for the selected month and does not compute its own calendar | 2026-10-22 (owner) | `01` 5.7 |
| DEC-007 | Users are created by an admin (original wording also planned Microsoft accounts; Microsoft is now skipped, see DEC-007a and DEC-038) | 2026-10-22 (owner) | `02` section 11 |
| DEC-034 | DIST total (144) is a distinct count from the database; active-distributor totals come from a **manual Excel upload** (`distaktif`) that may be stale or wrong. Both are shown as stored; mismatches go to a separate **Data Findings** sheet for validation | 2026-10-22 (owner) | `08-data-findings.md`, `DASH-009` |
| DEC-035 | A table name without the INA suffix belongs to ASW (so `v_sr_per_branch` is the ASW mapping); each principal has its own regions, branches and other data. Open remainder: `distaktif` (DEC-044) | 2026-10-22 (owner) | `03` 2.6 |
| DEC-036 | Admin sets roles when creating users and keeps full control afterwards: can change what a role is allowed, and switch features and filters on or off | 2026-10-22 (owner) | `02` section 10, SEC-014 |
| DEC-045 | AI Chat icon in the right panel; answers only from database data; no assumption or hallucination; pop-up chart/card/line/bar; pop-up hides but the result stays in chat and reopens | 2026-10-22 (owner) | `07-ai-chat.md` |
| DEC-041 | Data is **monthly**, refreshed every night; the current month is month-to-date as of the last refresh. For "month to date" questions the AI says only monthly periods exist (no "last month to date"). Near-real-time refresh is planned later | 2026-10-22 (owner) | `01` METRIC-018, `07` AI-026, `DASH-010` |
| DEC-044 | `distaktif` holds both INA and ASW regions now and will be split into separate tables later. Until then filter by each principal's region codes; after the split change only the principal registry | 2026-10-22 (owner) | `03` 2.6 |
| DEC-038 | Microsoft sign-in is **skipped** for now (no Microsoft administrator). Revisit when one is available | 2026-10-22 (owner) | `02` section 11, AUTH-009 |
| DEC-007a | Release 1 sign-in is admin-created username and password; users can change their password (supersedes the Microsoft part of DEC-007) | 2026-10-22 (owner) | `02` section 11 |
| DEC-040b | AI chat history is deleted after **60 days** | 2026-10-22 (owner) | `07` AI-035 |
| DEC-046a | AI behavior: confirms typos/wrong questions and suggests alternatives, answers after confirmation; answers from dashboard data; may do logical calculation; no free database queries and no tables outside the existing ones | 2026-10-22 (owner) | `07` AI-007, AI-008, AI-024 |
| DEC-031 | Every dashboard chart element has a tooltip with target, real value and % | 2026-10-22 (owner, in chat) | `01` section 7, `06` DASH-001 |
| DEC-032 | Growth dashboards always show a rolling 7 months ending at the selected (default current) month, with last year's same months | 2026-10-22 (owner, in chat) | `06` DASH-002, GROW-001 |
| DEC-033 | Four reference dashboards in scope: Sales Performance, Growth Sell In, Growth Sell Out, Daily Upload. Source table names supplied in `table_name` | 2026-10-22 (owner, files) | `06`, `03` section 4 |

## Blocking Phase 0 and 2

| ID | Question | Proposed default | Impact if wrong | Blocks | Status |
|---|---|---|---|---|---|
| DEC-001 | PostgreSQL schema verification | Table names and Tableau-derived columns are in `03-data-model.md` section 4. Still needed: `information_schema.columns` output, row grain of the performance view, join keys, region display order | Entire data layer | Phase 2 | Partly answered |
| DEC-002 | Are DIST and SE distinct? Which appear on cards vs header? | Treat as distinct until told otherwise | Wrong cards/totals | Phase 2 | Open |
| DEC-003 | Org hierarchy and roll-up | Region (RSM) > Branch (ASM) > Area (SPV) > Selling point (DIST) > Salesman (SE); achievement aggregated per level (`03` 4.4) | Wrong role achievement | Phase 2 | Proposed |
| DEC-004 | Achievement formulas, aggregation order, zero/missing target, denominator for people without data | Aggregate then divide; missing target = unavailable; missing-data people excluded from denominator but flagged | Wrong percentages | Phase 2 | Proposed |
| DEC-004a | Do IN/OUT mean Sell IN/Sell Out? Does exactly 100% count? | Yes, and yes (>=100%) | Wrong counts | Phase 2 | Proposed |
| DEC-006 | Business timezone and working-day semantics (as-of date, holidays, past/future periods, zero working days) | Business timezone confirmed by owner (placeholder `TBD`); past period 100%, future 0%, zero working days unavailable | Wrong progress, wrong colors | Phase 2 | Proposed |
| DEC-006a | Default date range on first load | Current calendar month to date | Confusing first view | Phase 2 | Proposed |

## Platform and operations

| ID | Question | Proposed default | Blocks | Status |
|---|---|---|---|---|
| DEC-008 | Hosting and approved image storage | Single container deployment; curated server-side asset library in v1 (no arbitrary uploads) | Phase 4 | Open |
| DEC-009 | Where app-owned config lives | Separate `app_config` schema in the same PostgreSQL, created by migrations (`03-data-model.md` section 3) | Phase 3 | Proposed |
| DEC-010 | Do filters persist across rotation or per dashboard? | Period filter shared; other filters per dashboard | Phase 5 | Proposed |
| DEC-012 | Kiosk display authentication | Dedicated long-lived read-only display token, revocable by admin | Phase 5 | Proposed |
| DEC-013 | UI language, number/date formatting, single vs multiple timezones across regions | Single business timezone; language confirmed by owner | Phase 1 | Open |
| DEC-014 | Data refresh cadence, staleness threshold, caching, read replica or materialized views | Query on demand with short server cache (60 s); stale after 2x source cadence | Phase 2 | Proposed |
| DEC-015 | Layouts/playlists scope: global, per role, per user, per display | Global layouts; one default playlist; per-display playlists later | Phase 4 | Proposed |
| DEC-016 | Rotation minimum duration and failure behavior | Minimum 5 s; on failure skip with notice | Phase 5 | Proposed |
| DEC-017 | Target displays (resolution, aspect, viewing distance) and minimum viewport | 1920x1080 and 1366x768 desktop; tablet-width reflow | Phase 1 | Open |
| DEC-018 | Tableau parity: sample periods and tolerance (workbook now provided) | 3 recent months incl. Oct-26; exact match after rounding | Phase 2 exit | Proposed |
| DEC-019 | Out-of-scope features: export, drill-through, annotations, alerts | All deferred past v1 | Phase 6 | Proposed |

## New decisions from the Tableau workbook, screenshots and data dictionary

| ID | Question | Evidence | Proposed default | Blocks | Status |
|---|---|---|---|---|---|
| DEC-023 | Growth bar colors: this-year bars green/yellow, last-year bars green/red. Exact rule | Tableau `ColorSellIN -1` and `LY` formulas | Observed behavior (`06` GROW-005) | Growth phase | Open |
| DEC-025 | Growth with zero last-year value; hide growth for in-progress month | Tableau uses `ZN` and a closing-month filter | "n/a" when last-year is 0; hide for current month | Growth phase | Proposed |
| DEC-026 | Daily Upload: row key (selling point vs city), day count, weekend/holiday, stock-upload variant, vacant rows | Screenshot 4 and workbook | Row = selling point; weekends neutral; stock variant as a toggle | Upload phase | Open |
| DEC-028 | SE definition: distinct `SLSNO`; exclude codes containing SOA/SOI? Use `FLAG_AKTIF`, `BranchActiveFlag`, `SLSTP`? | Workbook fields | Distinct `SLSNO`, active only, SOA/SOI excluded when owner confirms | Phase 2 | Open |
| DEC-029 | Multi-month range: denominators for role counts (entities x months), or single month only | Tableau formula | Allow month ranges; denominator entities x months | Phase 2 | Open |
| DEC-030 | Unit and currency of target/real (value or quantity), thousands format, abbreviation in tooltips | `TOTAL...` fields are integers; unit unknown | Show raw integers with thousands separators; unit per owner | Phase 2 | Open |

## Follow-ups created by the owner's answers

| ID | Question | Evidence | Proposed default | Blocks | Status |
|---|---|---|---|---|---|
| DEC-037 | Working day "today": whether today counts as elapsed. Tableau excludes today (18 of 27 on 22-Oct-2026) | Workbook, screenshot | Sum `networkeddays` for dates before today within the month; verify it reproduces 66.67% | Phase 2 | Proposed |

## Follow-ups from the AI chat, sign-in and findings requirements

| ID | Question | Proposed default | Blocks | Status |
|---|---|---|---|---|
| DEC-039 | Local account policy (now the only sign-in): minimum length, forced change, lockout, admin-only reset, MFA for admin accounts | 12+ characters, forced change, lockout after 5 failures, admin reset, optional TOTP MFA for admins | Phase 1 | Proposed |
| DEC-040 | AI remainder: who may read chats (retention is 60 days, confirmed), usage limits | Only the owner of a chat reads it; non-content usage counters kept | AI phase | Proposed |
| DEC-042 | AI reply language | Same language as the user's question (Bahasa Indonesia or English) | AI phase | Proposed |
| DEC-043 | Findings thresholds and severity (outlier ceiling, upload-gap days, difference tolerance), who validates, retention | Warning above 10% difference; upload gap 3 days; validators are users with `findings.manage` | Findings phase | Proposed |
| DEC-046 | Where can the app read the time of the last nightly refresh (`dataAsOf`)? A column, an ETL log table, or latest record date? | Read from an ETL/refresh metadata table if one exists; else show the latest month present | Phase 2 | Open |
| DEC-047 | AI gateway and models: 9Router, OmniRoute or OpenCode on a local server. Which upstream models may receive aggregated business data? Fully local model or cloud? Pin one model and disable fallback/compression? | Use an OpenAI-compatible gateway or local model server (not OpenCode as backend); pinned qualified model; fallback and compression off | AI phase | Open |
