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
| 10 | `RealSellInReg/Fes`, `RealSellOutReg/Fes` and their targets (unused). **Reg = regular products sold every day; Fes = festive products sold seasonally** (owner, DEC-086) | **Product groups:** achievement, mix and growth for Regular versus Festive, and the seasonal pattern | Yes (pace for Fes, DEC-091) |
| 11 | Quantities: `RealQtySLI` (Sell IN), `RealQtySLO` (Sell OUT), `TargetQtyEskaSLO` (Sell OUT target by quantity); `TargetValueEskaSLO` (Sell OUT target by value) | **Volume versus value:** Sell OUT quantity achievement, quantity growth against value growth, value per unit | Yes (unit and source checks, DEC-094, DEC-095) |
| 12 | Mapping: SE per distributor, customers (`CUSTNO` code, `CUSTNAME` name) | **Coverage and productivity:** customers per SE and per distributor, Sell OUT per SE and per customer (averages over the current mapping) | Yes (join key, snapshot caveat, DEC-096) |
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
| 7. Product groups: Regular and Festive | Which product group drives or drags the achievement, and how seasonal is it? | Achievement % for Regular and Festive (Sell IN and Sell OUT) per region; mix share of real versus mix share of target; growth against last year per group; shortfall by group; a 12-month seasonal view of Festive this year versus last year |
| 8. Volume and coverage | Are volumes and the field force growing with value? | Sell OUT quantity achievement per region; quantity growth against value growth; value per unit for Sell IN and Sell OUT against last year; customers per SE and per distributor; Sell OUT per SE and per customer (averages) |

Views 1 to 5, 7 and 8 use columns whose meaning is now confirmed and need the Sales Performance services plus monthly history; view 8 additionally waits for the schema check to confirm the quantity unit and the mapping join (DEC-094, DEC-096). View 6 uses the manual-upload and upload-log sources.

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
- `INS-015` `[C]` **Regular versus Festive is analysed** (owner): Regular (`Reg`) is the product group sold every day; Festive (`Fes`) is the group sold seasonally. Achievement is computed per group for Sell IN and Sell OUT from the group columns (`03-data-model.md` section 4.2).
- `INS-016` `[P]` **Mix.** Show each group's share of real value and of target value, and the gap between the two shares in points, so a region that sells more Festive than planned is visible even when the total looks fine.
- `INS-017` `[P]` **Seasonality.** A 12-month view of Festive real value, this year against last year, with the same-month last-year value as the reference. It describes the pattern in the data; it does not forecast the season.
- `INS-018` `[P]` **Pace for Festive.** The working-day pace assumes steady selling, which fits Regular products. Festive products sell in seasons, so the same pace can mislead. Default: apply the same rule to both groups (consistency) and show a note on Festive bars: "seasonal group, compared with the working-day pace"; for closed months both groups are judged against 100%. The owner may choose a different expectation for Festive (DEC-091).
- `INS-019` `[P]` **Group selector across the product.** Sales Performance, Growth, Executive Summary and Sales Detail get a product-group selector: All (default, the `TOTAL` columns), Regular or Festive. It swaps target and real consistently, never mixing groups (`DASH-014`, DEC-092).
- `INS-020` `[C]` **Meanings** (owner): SLI is Sell IN, the sale to the distributor; SLO is Sell OUT, the sale from the distributor to the customer. Quantity columns are the realised quantities; `TargetQtyEskaSLO` and `TargetValueEskaSLO` are the Sell OUT targets by quantity and by value.
- `INS-021` `[P]` **Volume versus value.** Per region and month: Sell OUT quantity achievement; quantity growth and value growth against last year side by side; value per unit for Sell IN and Sell OUT. Sell IN shows quantity without a percentage because no Sell IN quantity target exists.
- `INS-022` `[P]` **Coverage.** Customers per SE and per distributor, and Sell OUT per SE and per customer, computed from the current mapping and labelled "average, current mapping". A drill-down from distributor to SE to customer lists names only, with no figures (`PIVOT-017`).
- `INS-023` `[P]` Value per unit is labelled as such and never called price; changes are shown with the product-group split when the selector is set to Regular or Festive.
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

- Regular plus Festive equals the `TOTAL` columns for target and real (Sell IN and Sell OUT); mix shares sum to 100% for real and for target; group achievement equals hand calculation; the selector never mixes groups; a mismatch raises a `TYPE_TOTAL_MISMATCH` finding instead of being hidden.

- Quantity achievement equals `RealQtySLO / TargetQtyEskaSLO`; Sell IN shows no quantity percentage; value per unit equals value divided by quantity; coverage averages equal hand calculations over the mapping snapshot; zero quantity gives "n/a"; a quantity without value (or the reverse) raises `QTY_VALUE_MISMATCH`; `TargetValueEskaSLO` versus `TOTALTargetSellOut` differences raise `TARGET_SOURCE_MISMATCH`.

## 8. Open decisions

DEC-085 (this dashboard and its view order), DEC-096 (mapping join key, `FLAG_AKTIF`, `SLSTP`, `TEAM` meanings), DEC-089 (year start for YTD), DEC-090 (default thresholds), DEC-091 (pace expectation for Festive), DEC-092 (product-group selector across the product), DEC-094 (quantity unit), DEC-095 (`TargetValueEskaSLO` versus `TOTALTargetSellOut`). DEC-086 and DEC-087 are answered. See `decisions.md`.
