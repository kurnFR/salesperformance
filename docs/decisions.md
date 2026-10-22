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
| DEC-007 | Identity provider for users | Organization SSO via OIDC if available; otherwise local accounts behind HTTPS; dev uses a stub role switcher | Phase 4 | Open |
| DEC-008 | Hosting and approved image storage | Single container deployment; curated server-side asset library in v1 (no arbitrary uploads) | Phase 4 | Open |
| DEC-009 | Where app-owned config lives | Separate `app_config` schema in the same PostgreSQL, created by migrations (`03-data-model.md` section 3) | Phase 3 | Proposed |
| DEC-010 | Do filters persist across rotation or per dashboard? | Period filter shared; other filters per dashboard | Phase 5 | Proposed |
| DEC-011 | Row-level data scope per user (for example an RSM sees only their region) | No row-level scope in v1; all viewers see all regions | Phase 2 | Open |
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
| DEC-020 | Is exactly 70% yellow or red? | Owner: 70% to 99% yellow. Tableau: "> 70%" yellow | Inclusive (70% is yellow) | Phase 2 tests | Proposed |
| DEC-021 | Role-card "achieved" threshold: >= 100% or >= working-day progress | Screenshot: RSM IN 1 of 5 with max region 77%; Tableau has WD variants | >= working-day progress | Phase 2 | Open (important) |
| DEC-022 | Principals: INA and ASW. Scope of release 1? ASW has 6 regions (adds SUMATERA 3); ASW mapping table is `v_sr_per_branch` without suffix; no ASW active-distributor table listed | Data dictionary, Tableau parameters | INA only in release 1; `principalId` in all contracts | Phase 2 | Proposed |
| DEC-023 | Growth bar colors: this-year bars green/yellow, last-year bars green/red. Exact rule | Tableau `ColorSellIN -1` and `LY` formulas | Observed behavior (`06` GROW-005) | Growth phase | Open |
| DEC-024 | Sheet 2 region cards "30 / 31 (97%)": meaning | Likely active / total distributors from `distaktif`. Sheet 3 shows role cards instead | Active / total distributors per region; confirm sheet 3 | Growth phase | Open |
| DEC-025 | Growth with zero last-year value; hide growth for in-progress month | Tableau uses `ZN` and a closing-month filter | "n/a" when last-year is 0; hide for current month | Growth phase | Proposed |
| DEC-026 | Daily Upload: row key (selling point vs city), day count, weekend/holiday, stock-upload variant, vacant rows | Screenshot 4 and workbook | Row = selling point; weekends neutral; stock variant as a toggle | Upload phase | Open |
| DEC-027 | Working-day rule: today excluded; 6-day week; holidays; zero progress epsilon | 18 / 27 = 66.67% on 22-Oct-2026 suggests Mon to Sat, today excluded | Elapsed = working days strictly before today; verify with `dimdate` | Phase 2 | Proposed |
| DEC-028 | SE definition: distinct `SLSNO`; exclude codes containing SOA/SOI? Use `FLAG_AKTIF`, `BranchActiveFlag`, `SLSTP`? | Workbook fields | Distinct `SLSNO`, active only, SOA/SOI excluded when owner confirms | Phase 2 | Open |
| DEC-029 | Multi-month range: denominators for role counts (entities x months), or single month only | Tableau formula | Allow month ranges; denominator entities x months | Phase 2 | Open |
| DEC-030 | Unit and currency of target/real (value or quantity), thousands format, abbreviation in tooltips | `TOTAL...` fields are integers; unit unknown | Show raw integers with thousands separators; unit per owner | Phase 2 | Open |
