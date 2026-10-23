# Insight Dashboard: "Performance Drivers"

Related: [`PRD.md`](../PRD.md) · [`01-metrics-contract.md`](01-metrics-contract.md) · [`06-dashboard-catalog.md`](06-dashboard-catalog.md) · [`03-data-model.md`](03-data-model.md) · [`09-management-experience.md`](09-management-experience.md)

Tags: `[C]` owner-confirmed, `[R]` reference-observed, `[P]` proposed, `[O]` open.

## 1. Owner request and approach

- `INS-001` `[C]` The owner asked for **one new dashboard**, designed from **data already in this project that no dashboard uses yet**, to give users and management more insight into the business, and to point out what may have been missed.
- `INS-002` `[C]` Only existing approved tables and columns are used. No forecasts, no causes, no external data. Every figure is arithmetic on stored values, and every sentence on the page is generated from those figures by fixed templates, never invented.

## 2. Gap analysis: what the data can say that no dashboard says yet

The current dashboards answer "what is the achievement" and "how does it compare with last year". They do not answer "where is the missing value", "who stays behind", "is stock building up in the channel", "is the target itself realistic", or "how healthy is the distributor network". The table lists what is available and what is blocked.

| # | Data already in the project | Insight for management | Computable now? |
|---|---|---|---|
| 1 | Target and real by entity (only shown as %) | **Where is the missing value:** which regions, branches, SPVs and distributors account for most of the shortfall (Pareto) | Yes |
| 2 | Target and real, with the working-day pace | **Value behind pace:** real minus target pro-rated by working days, in value, not only in points | Yes |
| 3 | Sell IN and Sell OUT at the same distributor (shown separately) | **Channel stock signal:** sell-through % and Sell IN minus Sell OUT, over months | Yes (value) |
| 4 | Monthly history at every level (growth shown only by region) | **Persistence and momentum:** entities red for several closed months; improving or worsening | Yes |
| 5 | Status rule applied to all entities | **Status distribution:** how many entities are red, yellow, green, and the change from last month | Yes |
| 6 | Target and last year's real | **Target ambition:** is the target above last year's actual, by how much (explains many red bars) | Yes |
| 7 | Monthly sums from January (Tableau has a YTD sheet; the new product has none) | **Year to date:** YTD achievement, expected to date, growth vs last year's YTD | Yes (year start DEC-089) |
| 8 | `distaktif`: total, active, resigned per month | **Network health:** active % and resignations over months (manual upload, flagged) | Yes |
| 9 | Daily upload logs (grid only today) | **Reporting discipline:** upload compliance % by region and branch | Yes (eligible days DEC-026) |
| 10 | `RealSellInReg/Fes`, `RealSellOutReg/Fes` and their targets (unused) | **Mix:** regular versus the other type, share and growth | Blocked: meaning of "Fes" (DEC-086) |
| 11 | `RealQtySLI/SLO`, `TargetQtyEskaSLO`, `TargetValueEskaSLO` (unused) | **Volume versus value:** value per unit proxy, quantity achievement | Blocked: meaning of the columns (DEC-087) |
| 12 | Mapping: SE per distributor, customers (`CUSTNO`) | **Productivity and coverage:** Sell OUT per SE, customers per SE | Blocked: join keys, active flags (DEC-088) |
| 13 | `dimdate` week, quarter, holiday | Weekly and quarterly cuts | Later, needs day-level data (Marketing/SE) |

Deliberately not included: sales forecasts, "why" explanations, price effects, anything that needs data the project does not hold.

## 3. The dashboard

**Performance Drivers** (Sales department). One dashboard with **views** (tabs). Each view has its own link, so each can be a **playlist item** on the CEO TV. Filters shared by all views: principal buttons, month, hierarchy level (Region, Branch, SPV, Distributor) and, where it makes sense, a parent entity.

| View | Question it answers | Content |
|---|---|---|
| 1. Gap | Where is the missing value? | Pareto bars: top contributors to the shortfall at the chosen level, with cumulative share; switch between "gap to full target" and "gap to pace (value)"; one generated sentence, for example "5 distributors hold 62% of the shortfall" |
| 2. Channel | Is stock building up or draining? | Sell IN and Sell OUT lines over the rolling 7 months, net gap bars, sell-through % labels, lists of the largest positive and negative net gaps. Wording: "Sell IN minus Sell OUT", never "stock" |
| 3. Persistence | Who stays behind, who is turning around? | Heat map: entities by last 6 closed months in the rule's colors, streak count, momentum arrow |
| 4. Distribution and ambition | How wide is the problem, and are targets realistic? | Stacked counts red/yellow/green per region (and change from last month); target versus last year's real per region |
| 5. YTD | How is the year going? | YTD target, real, expected to date, achievement and growth vs last year's YTD, per region |
| 6. Network | Is the distributor base healthy and reporting? | Active % and resigned trend (flagged "manual upload"); upload compliance % |
| 7. Mix and volume (planned) | What is the type and unit mix? | Needs DEC-086 and DEC-087 |
| 8. Productivity (planned) | How productive is the field force? | Needs DEC-088 |

Views 1 to 5 use confirmed columns and need only the Sales Performance services plus monthly history. View 6 uses the manual-upload and upload-log sources. Views 7 and 8 are specified now and built when the owner confirms the semantics.

## 4. Requirements

- `INS-003` `[P]` **One dashboard, several views**, each deep-linkable (for example `#gap`) so the CEO can put any view in his rotation (`PLAY-009`).
- `INS-004` `[C]` Same tooltip contract as every dashboard (target, real, %, plus the view's own figure) (`DASH-001`); on the TV the same values appear as persistent labels (`DASH-012`).
- `INS-005` `[C]` Same status colors and rule as the rest of the product (RULE-001 to RULE-005); closed months are judged against 100%, the current month against the working-day pace.
- `INS-006` `[P]` **Generated sentences** ("so what" lines) come from fixed templates filled with computed numbers, in English and Bahasa Indonesia (`I18N-005`). No free text from a model on this page.
- `INS-007` `[P]` Every entity name and number links to the Sales Detail table pre-filtered (drill-down, `PIVOT-012`).
- `INS-008` `[P]` Scope applies to every view: a Regional Manager sees only their own region, including rankings, distributions and sentences ("5 distributors" means within the user's access).
- `INS-009` `[P]` Views fill the whole window and reflow between half and full screen (`UI-041`). Default is the half-screen layout.
- `INS-010` `[P]` Thresholds (chronic-streak length, momentum threshold, top N, sell-through bands) are admin settings with defaults; no number is hard-coded in the UI (DEC-090).
- `INS-011` `[P]` Closed-month history is read from the same approved tables; entities with no target in a month are shown as "n/a" and break a streak, never counted as red or green.
- `INS-012` `[P]` The AI chat gets matching tools (section 6) so every figure on this dashboard can also be asked in chat, with identical numbers.
- `INS-013` `[P]` Data caveats are visible where they apply: manual-upload sources, the in-progress month ("month to date"), and open data findings (`FIND-004`).
- `INS-014` `[P]` Export (`MGMT-007`) of any view's table and a snapshot of any view.

## 5. Metric definitions

Defined in `01-metrics-contract.md` section 12 (`METRIC-030` to `METRIC-040`): expected to date and value pace gap, shortfall share (Pareto), sell-through and net channel gap, status streak, momentum, status distribution, target ambition, YTD, upload compliance, and the planned mix, value-per-unit and productivity metrics.

## 6. AI tools and findings

- AI tools (read-only, scope-enforced, reusing these services): `getShortfallShare`, `getSellThrough`, `getStatusStreaks`, `getStatusDistribution`, `getTargetAmbition`, `getYtd`, `getNetworkHealth` (`07-ai-chat.md` section 3).
- New data findings (`08-data-findings.md`): Sell OUT far above Sell IN for a distributor over several months, a quantity with no value or the reverse (when the quantity columns are registered), and schema drift on any column this dashboard reads.

## 7. Acceptance and tests

- Shortfall shares sum to 100% across contributors; the Pareto cumulative line ends at 100%; sentences equal the numbers beside them.
- Net gap and sell-through equal hand calculations; zero Sell IN gives "n/a".
- Streaks count only closed months, break on "n/a", and ignore the in-progress month; momentum equals the hand-calculated difference.
- Status distribution counts equal the number of entities in the Sales Detail table for the same filters.
- YTD equals the sum of the monthly values; year boundaries and January are correct.
- Out-of-scope users get no data, no names and no counts from any view.
- Every view passes the four-viewport matrix and fills the window with no blank area (`UI-041`).

## 8. Open decisions

DEC-085 (this dashboard and its view order), DEC-086 (meaning of the type split "Reg" and "Fes"), DEC-087 (meaning of the quantity and Eska SLO columns), DEC-088 (join keys and active flags for productivity), DEC-089 (year start for YTD), DEC-090 (default thresholds). See `decisions.md`.
