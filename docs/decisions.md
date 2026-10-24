# Decision Log

Every open question has a **proposed default** so work can continue; the default applies only until the owner decides. Update status and date when a decision is made, and cite the DEC ID in PRs.

Status: `Confirmed` · `Proposed` (default in use, awaiting owner) · `Open` (blocked, no default) · `Superseded`.

## Needed before Phase 1 starts (owner)

**Nothing.** Every Phase 1 prerequisite has an answer (DEC-008, DEC-009, DEC-013, DEC-017, DEC-078, DEC-080). The remaining open items have proposed defaults and belong to later phases.

## Needed for Phase 2 and later (not blocking Phase 1)

- **DEC-081** the log's status values and time zone (checked by the schema check).
- **DEC-076** run the schema check and approve the mapping rows.
- **DEC-029** multi-month role-count denominators.
- **DEC-023, DEC-026** growth bar colors; Daily Upload details (build later).
- **DEC-082** ask the ETL to log the start and end of the load step (optional).
- **DEC-085 to DEC-099** Performance Drivers decisions (Phase 6); **DEC-093** which tables come from Excel versus SQL Server; **DEC-104 to DEC-106** Mapping Assistant (later).
- **DEC-054, DEC-075** Executive Summary details; last-month-to-date cutoff (Marketing).

## Confirmed

| ID | Decision | Confirmed | Where used |
|---|---|---|---|
| DEC-001 | The column names in the Tableau workbook are **exactly the PostgreSQL column names**; the workbook is the source for the column registry. Types and grain are confirmed by the automated schema check (DEC-076) | 2026-10-22 (owner) | `03` section 4 |
| DEC-002 | **DIST** = `sellingpointname`. **SE** = sales executive; name column `slsname` (key `slsno`) | 2026-10-22 (owner) | `01` METRIC-026, `03` 4.4 |
| DEC-005 | Color rule: if achievement >= working-day progress -> green (even below 70%); otherwise <70% red, 70% to <100% yellow, >=100% green. Equality is green. Yellow overrides red when behind pace but >=70% | 2026-10-22 (owner, in chat) | `01-metrics-contract.md` section 4 |
| DEC-005a | Working-day progress is capped to 0-100% on the server | 2026-10-22 | `01-metrics-contract.md` 5.7 |
| DEC-005b | Working-day progress comes from the working-day table/calendar, not sales achievement | Earlier (owner) | `01-metrics-contract.md` 5.7 |
| DEC-005c | Role card example: RSM (5), IN 20% / 1 = one of five counted as achieved. The threshold behind "achieved" is settled in DEC-021 | Earlier (owner) | `01-metrics-contract.md` 5.3 |
| DEC-007 | Users are created by an admin (original wording also planned Microsoft accounts; Microsoft is now skipped, see DEC-007a and DEC-038) | 2026-10-22 (owner) | `02` section 11 |
| DEC-007a | Release 1 sign-in is admin-created username and password; users can change their password (supersedes the Microsoft part of DEC-007) | 2026-10-22 (owner) | `02` section 11 |
| DEC-008 | Server: **Ubuntu 20.04 with Docker**, Node.js **v24.21.0**, PostgreSQL **12.22**. The app runs in Docker | 2026-10-22 (owner) | `02` section 15 |
| DEC-009 | Use the **existing PostgreSQL** configured in `.env` (`PG_HOST`, `PG_PORT`, `PG_DB`, `PG_USER`, `PG_PASSWORD`); no second database. One full-access user is accepted for now; the app adds its own schema `app_config` and opens business-table sessions read-only. A separate read-only user stays optional later | 2026-10-22 (owner) | `02` section 14, `.env.example` |
| DEC-011 | Data scope is per user and configurable: principal, region and other filters. Some users see one region, some several; only users granted full access see both principals. Roles and permissions are data managed by admins, not code | 2026-10-22 (owner) | `02` section 10, `03` section 3, `PRD` section 5 |
| DEC-013 | UI languages: **English and Bahasa Indonesia**; **default English**; admin can change the default language | 2026-10-22 (owner) | `04` section 9 |
| DEC-017 | Target display: **4K TV**. For the CEO the dashboard takes **half of the screen** (CCTV on the other half) for some dashboards and the **full screen** for others. The screenshots are the half-screen layouts | 2026-10-22 (owner) | `04` section 10, `UI-037` |
| DEC-020 | 70% up to 99% is yellow (70% inclusive) | 2026-10-22 (owner) | `01` RULE-009 |
| DEC-021 | Role cards use the **same rule as the bars**: an entity is "achieved" when achievement >= working-day progress (bar condition 1). The count is live: when working-day progress rises above an entity's achievement it stops counting (for example RSM IN goes from 1 to 0) | 2026-10-22 (owner) | `01` 5.3, `06` PERF-002 |
| DEC-022 | Principal is chosen in the web with buttons **ASW** and **INA**; both principals are in release 1. Access per principal is set by admin permissions, never hard-coded | 2026-10-22 (owner) | `06` DASH-004, `02` section 10 |
| DEC-024 | Active distributor % = `Total Aktif / TOTAL DIST` from table `distaktif` (sheet 2 region cards, for example 30 / 31 = 97%) | 2026-10-22 (owner) | `01` METRIC-016, `06` section 3 |
| DEC-027 | Working days come from the `dimdate` table (`networkeddays`); the app sums them for the selected month and does not compute its own calendar | 2026-10-22 (owner) | `01` 5.7 |
| DEC-028 | **SE and SPV are already separated in the dimension** (view family `v_sr_per_branch...`), so the SE count is a plain distinct count of `SLSNO` (excluding codes containing "SOI" for INA), as Tableau does | 2026-10-22 (owner) | `01` METRIC-027, `03` 4.4 |
| DEC-030 | Target and real values are `int8`; calculations use `numeric`; no currency symbol in the data (display separators per locale, optional unit label is a display setting) | 2026-10-22 (owner) | `01` METRIC-028, METRIC-029, DATA-005 |
| DEC-031 | Every dashboard chart element has a tooltip with target, real value and % | 2026-10-22 (owner, in chat) | `01` section 7, `06` DASH-001 |
| DEC-032 | Growth dashboards always show a rolling 7 months ending at the selected (default current) month, with last year's same months | 2026-10-22 (owner, in chat) | `06` DASH-002, GROW-001 |
| DEC-033 | Four reference dashboards in scope: Sales Performance, Growth Sell In, Growth Sell Out, Daily Upload. Source table names supplied in `table_name` | 2026-10-22 (owner, files) | `06`, `03` section 4 |
| DEC-034 | DIST total (144) is a distinct count from the database; active-distributor totals come from a **manual Excel upload** (`distaktif`) that may be stale or wrong. Both are shown as stored; mismatches go to a separate **Data Findings** sheet for validation | 2026-10-22 (owner) | `08-data-findings.md`, `DASH-009` |
| DEC-035 | A table name without the INA suffix belongs to ASW (so `v_sr_per_branch` is the ASW mapping); each principal has its own regions, branches and other data. `distaktif` is covered by DEC-044 | 2026-10-22 (owner) | `03` 2.6 |
| DEC-036 | Admin sets roles when creating users and keeps full control afterwards: can change what a role is allowed, and switch features and filters on or off | 2026-10-22 (owner) | `02` section 10, SEC-014 |
| DEC-038 | Microsoft sign-in is **skipped** for now (no Microsoft administrator). Revisit when one is available | 2026-10-22 (owner) | `02` section 11, AUTH-009 |
| DEC-039 | **MFA is configurable per user**: admin enables or disables it for each user who needs it. Remaining local-account policy values stay as proposed | 2026-10-22 (owner) | `02` AUTH-010, AUTH-011 |
| DEC-039a | MFA method is **TOTP** | 2026-10-22 (owner) | `02` AUTH-011 |
| DEC-040b | AI chat history is deleted after **60 days** | 2026-10-22 (owner) | `07` AI-035 |
| DEC-041 | Data is **monthly**, loaded by the ETL (nightly today); the current month is month-to-date as of the last completed refresh. For "month to date" questions the AI says only monthly periods exist (no "last month to date"). Near-real-time refresh is planned later | 2026-10-22 (owner) | `01` METRIC-018, `07` AI-026, `DASH-010` |
| DEC-044 | `distaktif` holds both INA and ASW regions now and will be split into separate tables later. Until then filter by each principal's region codes; after the split change only the principal registry | 2026-10-22 (owner) | `03` 2.6 |
| DEC-045 | AI Chat icon in the right panel; answers only from database data; no assumption or hallucination; pop-up chart/card/line/bar; pop-up hides but the result stays in chat and reopens | 2026-10-22 (owner) | `07-ai-chat.md` |
| DEC-046 | An **ETL** loads source data, refreshes materialized views and runs stored procedures, taking almost 3 hours. Dashboards must **auto-refresh** when data changes | 2026-10-22 (owner) | `02` section 13, `DASH-011` |
| DEC-046a | AI behavior: confirms typos/wrong questions and suggests alternatives, answers after confirmation; answers from dashboard data; may do logical calculation; no free database queries and no tables outside the existing ones | 2026-10-22 (owner) | `07` AI-007, AI-008, AI-024 |
| DEC-047 | The AI provider is **configurable in settings**: admin selects provider (9Router, OmniRoute, local model, Claude, Gemini, OpenAI/Codex, any OpenAI-compatible), model, and saves the API key; can switch any time | 2026-10-22 (owner) | `07` section 9, AI-037 to AI-047 |
| DEC-048 | The ETL refresh log is **`dwh_prod.etl_refresh_log`** (`process_name`, `start_time`, `end_time`, `status`, `message`). Newest row: `Refresh Materialized Views`, 2026-10-22 04:24:28.270 to 05:14:07.327, `SUCCESS`. `dataAsOf` = `end_time` of the newest `SUCCESS` row of the configured process | 2026-10-22 (owner) | `02` OPS-005, `03` 2.6 |
| DEC-048a | The ETL materialized-view refresh log **already exists in a log table**; the app reads it for the "data as of" time and the refresh signal (polling) | 2026-10-22 (owner) | `02` OPS-005 |
| DEC-049 | The encryption master key is entered by the administrator in the server's `.env`; the app refuses to start without it | 2026-10-22 (owner) | `02` SEC-015 |
| DEC-052 | In Sales Detail the **user chooses the columns**; the owner's example is only an example, not a fixed layout. All measures (Target Sell IN, Target Sell OUT, Real Sell IN, Real Sell OUT) are selectable | 2026-10-22 (owner) | `06` PIVOT-004 |
| DEC-057 | Regional Manager and RSM are the same role. CEO and General Manager see all regions; the Regional Manager sees own region(s) | 2026-10-22 (owner) | `09` section 2 |
| DEC-058 | Sales Detail (pivot): user picks a period range, dimension headers (region, branch, SPV, distributor) and fact values (target, real Sell IN, real Sell OUT), with a grand total | 2026-10-22 (owner) | `06` section 6 |
| DEC-059 | In the current tables targets exist only per selling point (distributor). SE-level monthly target and month-to-date sell-out per salesperson are planned to arrive with the **Marketing** module | 2026-10-22 (owner) | `03` 4.4, `06` PIVOT-017 |
| DEC-060 | **Item = product name with product code `pcode`**. It is not part of this project; it will come with the **Marketing dashboard** later | 2026-10-22 (owner) | `03` 2.8, `10` DEPT-005 |
| DEC-061 | Hierarchical dimensions can be drilled down through the hierarchy until SE, or until item if it exists | 2026-10-22 (owner) | `06` section 6.1, PIVOT-014 |
| DEC-065 | Departments to plan for, in order: **Sales, Marketing, Finance, HR**; Marketing is built next | 2026-10-22 (owner, accepted proposal) | `10` section 7 |
| DEC-069 | Departments such as **Marketing, Finance, HR and others** will be added later; the UI and access model plan for them now; a **department role can be added to each user** and enabled or disabled per user; the **department table does not exist yet** | 2026-10-22 (owner) | `10-departments-and-modules.md` |
| DEC-070 | The SE-level monthly target and sell-out data **already exist** in tables; they are improved and used later in the Marketing module. Table names and columns will be supplied then | 2026-10-22 (owner) | `03` 4.4, `10` DEPT-019 |
| DEC-071 | SE **real values exist per day** (day-level). The ETL still refreshes nightly; a daily/intraday snapshot is not created yet and can be created later | 2026-10-22 (owner) | `06` PROG-004, `02` OPS-011 |
| DEC-074 | New modules (Marketing, then others) are added **after the Sales project is finished and validated**; new features follow validation | 2026-10-22 (owner) | `10` DEPT-020, `05` later phases |
| DEC-077 | The whole ETL takes about 3 hours: about **2 hours of loading** plus about **50 minutes of materialized-view refresh**, and the materialized-view refresh is the final step. Data counts as ready when that process shows `SUCCESS` | 2026-10-22 (owner, with log evidence) | `02` OPS-005 |
| DEC-078 | The CEO TV runs **two windows** (CCTV viewer and the dashboard browser). The user switches the dashboard window between half and full by hand; **default is half**. The dashboard must **fill the window with no blank space, or auto-fit** (auto-fit preferred) | 2026-10-22 (owner) | `04` UI-037, UI-041 |
| DEC-079 | The CEO manages which dashboards rotate on his TV and how long each shows | 2026-10-22 (owner) | `04` PLAY-009 |
| DEC-080 | The ETL touches **only `dwh_prod`**. It runs on **Pentaho**; its sources are **Excel files and a SQL Server on another server**. The app's `app_config` schema is outside its scope | 2026-10-22 (owner) | `02` OPS-017 |
| DEC-083 | The TV is driven by a **mini PC over HDMI**, so a mouse works | 2026-10-22 (owner) | `04` DASH-012 |
| DEC-084 | The ETL **sometimes drops, truncates and adds columns** | 2026-10-22 (owner) | `02` OPS-017 to OPS-021 |
| DEC-086 | **Reg** = Regular, a product group sold every day. **Fes** = Festive, a product group sold seasonally. The owner wants their achievement analysed | 2026-10-22 (owner) | `11` INS-015 to INS-019, `01` METRIC-039 |
| DEC-087 | **SLI** = Sell IN (sale to the distributor); **SLO** = Sell OUT (sale from the distributor to the customer). `RealQtySLI` and `RealQtySLO` are realised quantities; `TargetQtyEskaSLO` is the Sell OUT target by quantity; `TargetValueEskaSLO` is the Sell OUT target by value; `RealSellInReg/Fes` are values | 2026-10-22 (owner) | `03` 4.2, `01` METRIC-042 |
| DEC-088 | `CUSTNO` is the **customer code** and `CUSTNAME` the customer name | 2026-10-22 (owner) | `03` 4.4, `11` INS-022 |
| DEC-094 | Quantities are **already converted to cartons** | 2026-10-22 (owner) | `01` METRIC-042 |
| DEC-095 | `TargetValueEskaSLO` is the **same measure** as `TOTALTargetSellOut`; column names are not consistent across tables, so the registry maps each measure per table | 2026-10-22 (owner) | `03` 4.3, field catalog |
| DEC-096 | `KODESCABANG` = `SELLINGPOINTCODE`; `KOTA` = city; `KODEAREA` = SPV (area) code. `FLAG_AKTIF`: `Y` active, `N` no longer active (resigned for an SE). `SLSTP` = type of salesperson (sales mix, sales office, taking order, others). `TEAM` = job position (sales executive, SPV, ...) | 2026-10-22 (owner) | `03` 4.2 to 4.4 |
| DEC-098 | Active flags come from master tables: the **customer** flag is defined in `fcustmst`; the **distributor** flag in `mscabang` (`Y` active, `N` not active) | 2026-10-22 (owner) | `03` 4.2 |
| DEC-100 | Some distributors or cities are not defined or not mapped, so their region is NULL. Today dashboards **exclude** NULL region while the detail sheet **includes all regions including NULL** | 2026-10-22 (owner) | `01` section 13 |
| DEC-101 | Unmapped values: dashboards exclude them with a visible reconciliation note; Sales Detail includes them as "(Unmapped)" with an "Include unmapped" switch (default on) | 2026-10-22 (owner agreed) | `01` section 13 |
| DEC-102 | Roles with all regions (CEO, General Manager) see unmapped values by default; regional roles do not; admin can change | 2026-10-22 (owner agreed) | `01` METRIC-048 |
| DEC-103 | **Later feature:** AI fills unmapped header dimension values and normalizes names from the nearest existing words in the database, never creating a new word; results go into a fallback (rename/normalize) table used when a refresh still has NULL; the data owner is informed (including email alerts) to fix the master source; if the owner does not fix it, the AI's best suggestion is used | 2026-10-22 (owner) | `12-mapping-assistant.md` |

## Blocking Phase 0 and 2

| ID | Question | Proposed default | Impact if wrong | Blocks | Status |
|---|---|---|---|---|---|
| DEC-003 | Org hierarchy and roll-up | Region (RSM) > Branch (ASM) > Area (SPV) > Selling point (DIST) > Salesman (SE); achievement aggregated per level (`03` 4.4) | Wrong role achievement | Phase 2 | Proposed |
| DEC-004 | Achievement formulas, aggregation order, zero/missing target, denominator for people without data | Aggregate then divide; missing target = unavailable; missing-data people excluded from denominator but flagged | Wrong percentages | Phase 2 | Proposed |
| DEC-004a | Do IN/OUT on the role cards mean Sell IN and Sell Out? | Yes | Wrong counts | Phase 2 | Proposed |
| DEC-006 | Business timezone used for "today" and the display date (working days themselves come from `dimdate`, DEC-027) | Asia/Jakarta unless the owner says otherwise; past month 100%, future month 0%, zero working days unavailable | Wrong progress near midnight | Phase 2 | Proposed |
| DEC-006a | Default period on first load | Current calendar month (month picker, month-to-date as of the last refresh) | Confusing first view | Phase 2 | Proposed |

## Platform and operations

| ID | Question | Proposed default | Blocks | Status |
|---|---|---|---|---|
| DEC-010 | Do filters persist across rotation or per dashboard? | Period filter shared; other filters per dashboard | Phase 5 | Proposed |
| DEC-012 | Kiosk display authentication | Dedicated long-lived read-only display token, revocable by admin | Phase 5 | Proposed |
| DEC-014 | Staleness threshold and caching (refresh itself is covered by DEC-046 and DEC-048) | Cache keyed by `dataVersion` (no fixed TTL needed); warn when no completed ETL run is seen for 36 hours | Phase 2 | Proposed |
| DEC-015 | Layouts/playlists scope: global, per role, per user, per display | Global layouts; one default playlist; per-display playlists later | Phase 4 | Proposed |
| DEC-016 | Rotation minimum duration and failure behavior | Minimum 5 s; on failure skip with notice | Phase 5 | Proposed |
| DEC-018 | Tableau parity: sample periods and tolerance (workbook now provided) | 3 recent months incl. Oct-26; exact match after rounding | Phase 2 exit | Proposed |
| DEC-019 | Out-of-scope features | Export and drill-down are **in scope** for management (`MGMT-004`, `MGMT-007`). Annotations and push alerts stay deferred | Phase 6 | Proposed |

## New decisions from the Tableau workbook, screenshots and data dictionary

| ID | Question | Evidence | Proposed default | Blocks | Status |
|---|---|---|---|---|---|
| DEC-023 | Growth bar colors: this-year bars green/yellow, last-year bars green/red. Exact rule | Tableau `ColorSellIN -1` and `LY` formulas | Observed behavior (`06` GROW-005) | Growth phase | Open |
| DEC-025 | Growth with zero last-year value; hide growth for in-progress month | Tableau uses `ZN` and a closing-month filter | "n/a" when last-year is 0; hide for current month | Growth phase | Proposed |
| DEC-026 | Daily Upload: row key (selling point vs city), day count, weekend/holiday, stock-upload variant, vacant rows | Screenshot 4 and workbook | Row = selling point; weekends neutral; stock variant as a toggle | Upload phase | Open |
| DEC-029 | Multi-month range: denominators for role counts (entities x months), or single month only | Tableau formula | Allow month ranges; denominator entities x months | Phase 2 | Open |

## Follow-ups created by the owner's answers

| ID | Question | Evidence | Proposed default | Blocks | Status |
|---|---|---|---|---|---|
| DEC-037 | Working day "today": whether today counts as elapsed. Tableau excludes today (18 of 27 on 22-Oct-2026) | Workbook, screenshot | Sum `networkeddays` for dates before today within the month; verify it reproduces 66.67% | Phase 2 | Proposed |

## Follow-ups from the AI chat, sign-in and findings requirements

| ID | Question | Proposed default | Blocks | Status |
|---|---|---|---|---|
| DEC-040 | AI remainder: who may read chats (retention is 60 days, confirmed), usage limits | Only the owner of a chat reads it; non-content usage counters kept | AI phase | Proposed |
| DEC-042 | AI reply language | Same language as the user's question (Bahasa Indonesia or English) | AI phase | Proposed |
| DEC-043 | Findings thresholds and severity (outlier ceiling, upload-gap days, difference tolerance), who validates, retention | Warning above 10% difference; upload gap 3 days; validators are users with `findings.manage` | Findings phase | Proposed |
| DEC-050 | Data-sharing acknowledgement when a non-local AI provider is activated: wording, who may acknowledge | Admin acknowledges; stored in the audit log | AI phase | Proposed |
| DEC-051 | Optional secondary AI profile used only when the active one is down (`AI-047`) | Off by default | AI phase | Open |
| DEC-053 | Export scope: formats (Excel, CSV, PDF/image), row limit, who may export | Excel, CSV and PDF/image snapshot; row limit 50,000; capability `export.use` per role; no scheduled delivery in v1 | Phase 6 | Proposed |
| DEC-054 | Executive Summary content and default landing page per role | As in `09-management-experience.md` section 3 and 4 | Phase 2 | Proposed |
| DEC-055 | Alert and delivery channels (in-app only, email, WhatsApp) | In-app Needs-attention list only in v1 | Phase 7 | Open |
| DEC-056 | Derived management metrics (pace gap, shortfall, needed per remaining working day) | As in `01` section 9 | Phase 2 | Proposed |
| DEC-062 | Deepest hierarchy level per role (depth cap) | Every role may open all levels it has scope for; admin can cap per role | Phase 6 | Proposed |
| DEC-063 | Department list lives in the app's own `app_config.departments` (admin-managed), not in the business database | Yes, as in `03` section 3 | Phase 1 | Proposed |
| DEC-064 | Visibility of planned departments: "Coming soon" only to users who have them enabled (admins see all) | As described in `10` section 4 | Phase 1 | Proposed |
| DEC-066 | Sensitive departments (Finance, HR): off by default for everyone, separate export and AI capabilities, audited views | As in `10` section 6 | Before Finance/HR go active | Proposed |
| DEC-067 | Company summary across departments for CEO and General Manager | Later, once two departments are active | Later | Open |
| DEC-072 | SE achievement uses SE sell-out against an SE sell-out target; Sell IN is not an SE-level measure | Yes | Marketing | Proposed |
| DEC-073 | When SE sell-out sums differ from the distributor's Real Sell OUT (different sources): show "Unassigned / difference" and raise a data finding, never force equality | Yes | Marketing | Proposed |
| DEC-075 | Last-month-to-date cutoff for day-level sources: same calendar day or same working day? | Same working day (matches the working-day pace); show the cutoff date | Marketing | Proposed |
| DEC-076 | Verification of types, grain, regions and ETL log: run `pnpm schema:check` (Phase 1 script) with the read-only user and approve the report. This specification environment cannot reach the database | Owner or technical administrator runs it once Phase 1 provides the script | Phase 2 | Proposed |
| DEC-081 | Status values and time zone of `dwh_prod.etl_refresh_log` (running or failed rows, null `end_time`, local or UTC) | Checked by the schema check (DEC-076); a newer `start_time` without a newer successful `end_time` means "refresh in progress" | Phase 2 | Proposed |
| DEC-082 | Ask the ETL (Pentaho) to log the start and end of its load step (not only `Refresh Materialized Views`) so the maintenance window is exact; Pentaho jobs can write their own log tables | Estimate the window from log history until then | Phase 2 | Proposed |
| DEC-085 | The one new dashboard is **Performance Drivers** (views: Gap, Channel, Persistence, Distribution and ambition, YTD, Network, Product groups Regular and Festive, Volume and coverage) | As in `11` | Phase 6 | Proposed |
| DEC-089 | First month of the year for YTD (calendar or fiscal) | January | Phase 6 | Proposed |
| DEC-090 | Default thresholds: chronic streak 3 closed months; momentum 2 points; top N 5; sell-through bands | As listed | Phase 6 | Proposed |
| DEC-091 | Pace expectation for Festive (seasonal) products: the working-day pace can mislead for a seasonal group | Same rule for both groups with a visible note; closed months against 100%; owner may define a seasonal expectation | Phase 6 | Proposed |
| DEC-092 | A product-group selector (All, Regular, Festive) on Sales Performance, Growth, Executive Summary and Sales Detail | Yes, All by default | Phase 2 | Proposed |
| DEC-093 | Which `dwh_prod` tables are loaded from Excel (manual) and which from SQL Server? (`distaktif` is a manual Excel upload) | Label known Excel tables; add findings checks for staleness | Phase 2 | Open |
| DEC-099 | Which columns of the mapping view carry the customer flag (from `fcustmst`), the distributor flag (from `mscabang`) and the SE active or resigned flag? Are `fcustmst` and `mscabang` available in `dwh_prod`, or only visible through the view? | Use only view columns (expected `FLAG_AKTIF` for the customer, `BranchActiveFlag` for the distributor); the schema check lists values per column; SE active flag stays optional until its source is known | Phase 6 | Open |
| DEC-104 | Mapping Assistant approval policy: grace period before auto-apply, confidence threshold, and the impact limit above which a person must approve | 14 days; high confidence only; entries above 1% of a region's total always need a person | Mapping Assistant | Proposed |
| DEC-105 | History policy: apply a fallback from the approval month forward; backfill earlier months only with explicit approval and a restatement note | As described | Mapping Assistant | Proposed |
| DEC-106 | Email for data-owner alerts: SMTP server and sender in `.env`, the contact list per finding category, digest frequency, escalation days | Weekly digest, immediate message for large impacts, escalate after 7 days | Mapping Assistant | Open |
