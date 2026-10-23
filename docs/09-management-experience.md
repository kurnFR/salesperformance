# Management Experience Review (CEO, General Manager, Regional Manager)

Related: [`PRD.md`](../PRD.md) · [`06-dashboard-catalog.md`](06-dashboard-catalog.md) · [`01-metrics-contract.md`](01-metrics-contract.md) · [`07-ai-chat.md`](07-ai-chat.md)

Tags: `[C]` owner-confirmed, `[P]` proposed, `[O]` open.

## 1. Review: what the specification lacked for management use

The first specification reproduced the Tableau sheets faithfully. That serves analysts who already know where to look. Executives ask different questions and have little time. Gaps found:

| # | Gap | Effect on management | Addressed by |
|---|---|---|---|
| G1 | No single "how are we doing and where is the problem" page | CEO must open several sheets to answer one question | MGMT-001, MGMT-002 |
| G2 | No defined drill-down from region to distributor | A red region cannot be explained without leaving the dashboard | MGMT-004, `PIVOT-*` |
| G3 | Only percent achievement. No gap in value, pace gap or what is still needed | A percent does not say how much is missing or how hard the rest of the month is | MGMT-005 |
| G4 | Export and sharing were deferred (DEC-019) | Managers need Excel and PDF for meetings and WhatsApp/email | MGMT-007, MGMT-008 |
| G5 | No exception view (who is furthest behind, best movers) | Attention goes to the wrong places | MGMT-006 |
| G6 | No metric definitions visible in the product | Disputes about "how is this calculated" | MGMT-011 |
| G7 | No mobile-first layout | Regional managers read on phones | MGMT-012 |
| G8 | Scope exists, but no default landing per role | A regional manager lands on the national view | MGMT-003 |
| G9 | No saved views | Repeated filter setup | MGMT-009 |
| G10 | Alerts deferred | Problems found by chance | MGMT-010 |
| G11 | AI chat has no guidance for busy users | Blank chat box | MGMT-014 |
| G12 | No adoption measure | Cannot tell whether management uses it | MGMT-018 |

What is already strong for management: working-day pace coloring, data as-of time, data findings, scope per role, presentation mode, and a grounded AI.

## 2. Management personas (`[P]`; Regional Manager = RSM is confirmed in DEC-057)

| Persona | Scope (set by admin) | Typical questions | Device | Default landing |
|---|---|---|---|---|
| CEO | All principals, all regions | Are we on pace this month? Which region is the problem? Better or worse than last year? | Phone, meeting-room screen | Executive Summary |
| General Manager | All principals (or one), all regions | Which branches and SPVs drag the region? Who is furthest behind pace? | Laptop, phone | Executive Summary, then Sales Detail |
| Regional Manager (= RSM, confirmed) | Own region(s) | Which branches, SPVs and distributors in my region are behind? What do I need per day to reach 100%? | Phone, laptop | Regional view (own region) |
| ASM / SPV (optional) | Own branch or area | Which distributors need a visit today? | Phone | Regional view, filtered |

All of this reuses the access model: a persona is just a role with a scope and a default landing (`MGMT-003`). Nothing is hard-coded.

## 3. Requirements

- `MGMT-001` `[P]` **Executive Summary page**, one screen, built from existing metrics:
  1. Headline strip: Sell IN % and Sell Out % against working-day progress (the pace), for the user's scope, per principal.
  2. Pace gap and status color using the confirmed rule (RULE-001 to RULE-005).
  3. Region tiles (status color, achievement %, pace gap), sorted by gap.
  4. **Needs attention:** the regions, then branches and SPVs, furthest behind pace.
  5. **Movers:** largest growth up and down against the same month last year (closed months only; the in-progress month is labelled not like-for-like).
  6. **Data health:** data as-of time and the count of open data findings that affect the user's scope.
- `MGMT-002` `[P]` **Regional Manager view:** the same page scoped to the user's region(s), with branches, SPVs and distributors ranked by pace gap.
- `MGMT-003` `[P]` **Default landing per role**, chosen by the admin on the role (for example Executive Summary for CEO and GM, regional view for RSM).
- `MGMT-004` `[P]` **Drill-down** through the whole hierarchy (Region > Branch > SPV > Distributor, then SE and item where data exists, `PIVOT-014`) by clicking a tile, bar or row, with a breadcrumb and the same tooltip (target, real, %). Each level is the Sales Detail service filtered to the clicked item; scope always applies.
- `MGMT-019` `[P]` **Depth cap per role:** the admin sets the deepest hierarchy level a role may open (for example the CEO to Distributor, an SPV to SE), so individual-level figures are shown only to roles that should see them (`PIVOT-019`).
- `MGMT-005` `[P]` **Derived metrics**, defined in `01-metrics-contract.md` section 9: pace gap, shortfall to target, and "needed per remaining working day to reach 100%". These are arithmetic on existing values, not forecasts, and they are labelled that way.
- `MGMT-006` `[P]` **Rankings and exceptions:** top N and bottom N by achievement, pace gap or growth, at any level the user may see; default N = 5, configurable.
- `MGMT-007` `[P]` **Export and share:** Excel and CSV for the Sales Detail table and ranking lists; PDF or image snapshot of the Executive Summary and any dashboard for meetings. Files carry the filters, scope note and data as-of time. Capability `export.use`, assignable per role; limited to the user's scope; row limit with a clear message. Scheduled email or WhatsApp delivery is later (DEC-055).
- `MGMT-008` `[P]` **Shareable links** that carry the filters. A recipient outside the scope sees "not available for your access", never the data.
- `MGMT-009` `[P]` **Saved views** per user (filters, Sales Detail layout), with one default view.
- `MGMT-010` `[O]` **Alerts:** a notification list inside the app for "region below pace", "data stale", "critical finding". Email or WhatsApp channels need a decision (DEC-055). Until then, the Needs-attention list serves this purpose.
- `MGMT-011` `[P]` **Metric glossary:** every metric label has an info icon showing definition, formula, source table (name only), and as-of time. A glossary page is generated from `01-metrics-contract.md` so the product and the contract cannot drift.
- `MGMT-012` `[P]` **Phone and tablet layout** for the Executive Summary and Regional Manager view: single column of cards, large touch targets, numbers readable without zoom, no horizontal page scroll.
- `MGMT-013` `[O]` Indonesian language option and Indonesian number format; value units such as juta and miliar for large numbers (DEC-013, DEC-030).
- `MGMT-014` `[P]` **AI starter questions** by role shown as buttons in the AI chat (for example "Which region is furthest behind pace?", "Which SPVs in my region are below 70%?"). Each maps to an existing tool and is covered by the golden test set.
- `MGMT-015` `[P]` **Comparison basis switch:** against target (default), against last month (full month), against the same month last year, with the like-for-like label where the in-progress month is involved.
- `MGMT-016` `[P]` **Meeting mode:** presentation mode (exists) plus a print-friendly layout.
- `MGMT-017` `[P]` **Speed:** the Executive Summary reaches first useful content within 2.5 seconds on an ordinary office connection; results are cached by data version (`02-architecture.md` section 13).
- `MGMT-018` `[P]` **Adoption analytics:** privacy-safe counts of active users, pages and exports per role per week, visible to admins, to show whether management uses the product.

## 4. Executive Summary layout (`[P]`, desktop; phone stacks the blocks)

```
+------------------------------------------------------------------------------+
| [ASW] [INA]                      Oct-26 (month to date)   Data as of: <time> |
+------------------------------------------------------------------------------+
| SELL IN  72%  vs pace 67%  (+5 pt)  [green]   | SELL OUT 18% vs pace 67% [red]  |
+------------------------------------------------------------------------------+
| Region tiles:  JAWA 1 [..]  JAWA 2 [..]  PULAU 1 [..]  SUMATERA 1 [..]  ...    |
+----------------------------------+-------------------------------------------+
| NEEDS ATTENTION (furthest behind)| MOVERS vs same month last year            |
|  1. region / branch  gap  need/day|  up:   ...        down: ...               |
+----------------------------------+-------------------------------------------+
| Data health: 3 open findings (1 critical)  [open]        [Export PDF] [Share]  |
+------------------------------------------------------------------------------+
```

Numbers in the sketch are layout placeholders, not data.

## 5. Where this lands in the roadmap

- Phase 2: Executive Summary, Regional Manager view and drill-down (they reuse the Sales Performance services).
- Phase 3: glossary, default landing per role, saved views.
- Phase 6: Sales Detail (pivot) with export, ranking lists, shareable links.
- Phase 7: AI starter questions; alerts if a channel is approved.
- Phase 8: adoption analytics, performance budget checks.

## 6. Tests

- Executive Summary numbers equal the Sales Performance service output for the same scope and period.
- A regional manager never sees another region's tiles, rankings, exports or shared links.
- Derived metrics match hand calculations, including zero remaining working days and zero target.
- Export content equals the on-screen table, with filters and as-of note.
- Phone layout passes the responsive and accessibility checks in `04-ui-ux-spec.md`.
