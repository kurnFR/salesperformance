# Sales Performance Dashboard — Product Requirements Document

**Status:** Draft / living specification  
**Repository:** https://github.com/kurnFR/salesperformance  
**Product:** Interactive web replacement for the existing Tableau Sales Performance dashboard

## 1. Executive summary

Build a maintainable, responsive BI web application backed by the existing PostgreSQL database. Reproduce the supplied dashboard as the first view, then add configurable layouts, filters, themes, dashboard playlists, and administration incrementally. The app must work in current Firefox and Chrome, adapt to browser viewport changes, and support light and dark themes.

The owner will provide database tables and business definitions later. Until then, do not guess production schema, create business tables, or fabricate production metrics. Use typed contracts and clearly marked mock fixtures.

## 2. Goals and non-goals

### Goals
- Replace the Tableau dashboard with an interactive web app using the existing PostgreSQL source of truth.
- Match the reference: Start Date / End Date filters, role KPI cards, region navigation, Sell IN / Sell Out achievement bars, working-day progress, and display date.
- Keep metric definitions consistent and testable.
- Allow authorized editors to add/remove widgets, drag and resize items, edit text, adjust layouts, and configure background images.
- Let admins configure achievement thresholds/colors without code changes.
- Provide an admin-managed dashboard playlist that rotates automatically, defaulting to 10 seconds per dashboard.
- Build each sidebar dashboard incrementally using reusable components and a shared design system.

### Non-goals for the first release
- Replacing PostgreSQL or changing its business schema before it is provided.
- Guessing joins, column names, KPI formulas, or the meaning of role labels.
- Building a general-purpose arbitrary SQL editor.
- Implementing every future dashboard in the first milestone.

## 3. Users and permissions

- **Viewer:** View dashboards, change permitted filters, inspect chart values, and use presentation mode if enabled.
- **Editor:** Add/remove supported widgets, move/resize items, edit text and appearance, preview, and save layouts.
- **Administrator:** Manage dashboards, role access, achievement rules, backgrounds, playlists, dashboard order, and rotation duration.
- **Technical administrator:** Configure secrets, deployment, PostgreSQL connectivity, and operational monitoring.

Enforce permissions on the server, not only by hiding frontend controls. Select the authentication provider after repository/deployment discovery.

## 4. Reference dashboard requirements

### Header and filters
- Start Date and End Date filters with validation; Start Date cannot be after End Date.
- Show business-local display date and totals for RSM, ASM, SPV, and SE as defined by the approved business model.
- Filter changes update all dependent cards and charts consistently without full-page reload.
- Provide reset filters, loading, empty, stale-data, and error states.

### Role KPI cards
- Show role name, headcount, IN achievement percentage/count, and OUT achievement percentage/count.
- Owner-provided example: RSM (5) means five RSMs in scope. IN 20% / 1 means one of five RSMs has achievement >=100%, or 20%.
- Validate whether IN means Sell IN and OUT means Sell Out; define missing-data and denominator behavior.
- The image shows DIST cards while the header totals include SE. Do not assume DIST and SE are the same; confirm this with the owner.

### Region achievement charts
- Region tabs/sections (example: INA JAWA 1, INA JAWA 2, INA PULAU 1, INA SUMATERA 1, INA SUMATERA 2) are data/config driven, not hardcoded.
- Show paired Sell IN and Sell Out bars, percentage labels, accessible tooltips, and an accessible table/summary alternative.
- Support mouse and keyboard interaction, responsive resizing, and values above 100% if business rules permit.
- No production metric values may be hardcoded in UI components.

### Working-day progress
- The circular indicator represents elapsed working-day progress, not sales achievement. Example: 66.67%.
- Calculate from the approved working-day calendar/table and selected period/current business date. Do not substitute calendar days unless explicitly approved.
- The API should return the authoritative value and calculation metadata. Define weekends, holidays, date boundaries, timezone, and zero-working-day behavior.
- Label it clearly, e.g. “Working-day progress,” and show its as-of date.

## 5. Functional requirements

### Sidebar and navigation
- Collapsible sidebar with accessible toggle and modular dashboard sections.
- New dashboards can be added one at a time without rewriting the shell.
- Charts resize correctly when the sidebar collapses, the browser resizes, or the display changes orientation.
- Provide dashboard title, optional description, and last-updated timestamp.

### Filters and state
- Support global, dashboard-level, and widget-level filters with explicit scope.
- Validate inputs on both client and server; parameterize all database queries.
- Cancel stale requests and prevent out-of-order responses from overwriting newer results.
- Keep URL state shareable when appropriate and safe; provide a reset action.

### Flexible dashboard editor
- Separate View and Edit modes. In View mode, chart interactions remain enabled but layout drag/resize is disabled.
- Grid-based drag/drop, resizing, snapping, spacing, and responsive breakpoints.
- Initial widget registry: KPI card, bar chart, text/heading, image/background; later widgets must plug into the same registry.
- Add, remove, duplicate, move, resize, reorder, rename, and configure supported widgets.
- Text blocks support safe limited formatting; never render arbitrary unsanitized HTML.
- Background image controls: choose/upload approved asset, fit/position, overlay/opacity, and contrast. Validate type/size and use approved asset storage.
- Save, preview, cancel/revert, unsaved-change warning, versioned layout configuration, and validation.
- Store typed configuration JSON, never executable code or raw SQL.

### Theme and responsive UX
- Light, dark, and system-preference modes using shared design tokens.
- Current stable Firefox and Chrome desktop support; responsive behavior for smaller screens as well.
- Container-aware chart resizing and no unnecessary page-wide horizontal scrolling.
- Presentation/kiosk mode maximizes chart space and hides editing controls.
- Honor reduced-motion preferences and ensure readable contrast in both themes.

### Dashboard rotation
- Admin selects dashboards, ordering, enabled state, and duration per item.
- Default duration is 10 seconds; validate configurable durations and enforce a sensible minimum.
- Controls: start, pause, resume, next, previous, exit.
- Use one rotation controller/state machine; clean up timers/listeners and prevent overlapping intervals.
- Do not rotate while editing or while unsaved changes exist.
- Define visible behavior for a dashboard load failure; do not leave users staring at a blank screen.
- Persist playlists server-side after the storage model is approved.

### Achievement color settings
- Configure thresholds, semantic colors, and rule priority in admin settings; never hardcode these rules in chart components.
- **Owner-confirmed rule** (see section 17.8 for the full contract and test table). Two conditions are evaluated in a fixed order:
  1. **Condition 1 (working-day pace):** if achievement >= working-day progress, the bar is **green**. Equality counts as green. This applies even when achievement is below 70%.
  2. **Condition 2 (achievement band)**, evaluated only when condition 1 did not produce green (achievement < working-day progress):
     - achievement < warning minimum (default 70%): **red**
     - warning minimum <= achievement < 100%: **yellow**
     - achievement >= 100%: **green** (safe fallback; unreachable once working-day progress is capped at 100%)
- Condition 1 can only produce green. A "red" result from the working-day comparison is never final; it falls through to condition 2. Example: working-day progress 80% and achievement 75% is **yellow**, not red.
- Working-day progress is capped to the range 0-100% by the server, so the rule is deterministic.
- Validate threshold ordering/ranges and contrast; provide preview, restore defaults, safe fallback, and audit/version history where practical.
- Color must not be the only status signal; include labels/icons or text descriptions.

## 6. Metric definitions to approve

| Metric | Proposed definition | Owner must confirm |
|---|---|---|
| Sell IN achievement | Actual Sell IN / approved Sell IN target | Source fields, aggregation, zero/missing target |
| Sell Out achievement | Actual Sell Out / approved Sell Out target | Source fields, aggregation, zero/missing target |
| Role headcount | Count of people in role and selected scope | Active status and effective dates |
| IN count / % | People with approved IN achievement >=100%; count / role headcount | IN meaning, denominator, missing values |
| OUT count / % | People with approved OUT achievement >=100%; count / role headcount | OUT meaning, denominator, missing values |
| Working-day progress | Elapsed working days / total working days in selected period | Calendar source, holidays, timezone, inclusive boundaries |
| Region | Approved region grouping and display order | Keys, labels, hierarchy |
| Display date | Current date in business timezone | Business timezone |

Until schema and sample calculations are supplied, implement typed service interfaces and mock fixtures only. Do not apply SQL DDL or alter existing business tables.

## 7. Recommended exact package stack

Inspect the repository first and reuse sound existing conventions. Do not rewrite the app just to match this proposal or install everything in one commit.

### Frontend
- Node.js current LTS; pin runtime and package manager in docs/CI.
- Vite + React + TypeScript for a fast, typed frontend.
- Tailwind CSS for responsive styling and design tokens.
- shadcn/ui with Radix UI primitives for accessible, repository-owned UI components.
- lucide-react for icons.
- React Router for dashboard/sidebar routing.
- TanStack Query for API state, caching, invalidation, and request cancellation.
- Zustand for small client-only state (editor selection, theme UI state, rotation controller); do not duplicate server state.
- React Hook Form + Zod for validated settings/editor forms.
- Apache ECharts + echarts-for-react for interactive BI charts.
- react-grid-layout for draggable/resizable grid layouts, isolated behind an adapter so the library can be replaced if needed.
- date-fns for date utilities; authoritative working-day calculations remain on the server.
- ky or a shared native-fetch wrapper for typed API calls; choose one and standardize.
- With Vite, use a small theme provider rather than adding a Next.js-specific theme package.

### Backend and database
- Fastify + TypeScript for a modular HTTP API.
- Existing PostgreSQL database remains the source of truth.
- pg driver for PostgreSQL connectivity.
- Drizzle ORM + drizzle-kit for typed SQL and explicit migrations if app-owned tables are approved. If the repo already has a sound data-access layer, retain it instead of introducing a second ORM.
- Zod at request, response, and configuration boundaries.
- Pino structured logging with secret/data redaction.
- @fastify/helmet, @fastify/cors, and @fastify/rate-limit configured for deployment.
- @fastify/swagger and @fastify/swagger-ui for API contracts where useful.
- Parameterized SQL, least-privilege credentials, bounded results, query timeouts, and connection pooling.

### Quality and delivery
- pnpm workspaces if frontend and API share this repository; otherwise follow existing project conventions.
- Vitest, @testing-library/react, @testing-library/user-event, and jest-axe for unit/component/accessibility tests.
- Playwright for browser E2E tests in Chromium and Firefox.
- ESLint, typescript-eslint, Prettier, and TypeScript strict mode.
- CI: frozen-lockfile install, lint, typecheck, unit tests, build, and required E2E checks.

Verify the currently maintained package versions and compatibility when implementation starts; do not blindly copy version numbers into the PRD.

## 8. Architecture and proposed API

Prefer a modular monolith over microservices. Adapt this suggested structure to the actual repository rather than forcing a rewrite:

    apps/web/                 React + Vite UI
    apps/api/                 Fastify API
    packages/contracts/       Shared DTOs and Zod schemas
    packages/dashboard-core/  Metric types, layout schema, threshold evaluation
    packages/config/          Shared tooling config, if useful

Keep route handlers thin. Separate authorization, services/business rules, repository/query code, contracts, and UI. Never put SQL in React components or duplicate working-day/metric formulas in multiple layers. Never expose database credentials to the browser.

Proposed API contracts (not claims that these endpoints already exist):
- GET /api/health
- GET /api/dashboards
- GET /api/dashboards/:dashboardId
- GET /api/sales-performance/summary?startDate=&endDate=&regionId=
- GET /api/sales-performance/regions?startDate=&endDate=&regionId=
- GET /api/calendar/working-day-progress?startDate=&endDate=&asOfDate=
- GET/PUT /api/settings/achievement-rules (admin only)
- GET/POST/PUT /api/playlists (admin only)
- POST /api/dashboards/:id/layout (editor permission required)
- Asset list/upload endpoints only after storage is approved.

Validate requests/responses and return safe, consistent errors. Metric data should include values, labels, units, as-of timestamps, and working-day progress metadata.

## 9. Logical configuration models

These are logical models only, not approved SQL tables:
- DashboardDefinition: identity, name, description, route key, visibility, layout version, publication state.
- WidgetDefinition: widget type, title, allowlisted data binding key, validated options, grid position/size, visibility.
- DashboardLayoutVersion: dashboard, version, validated layout JSON, editor, timestamp.
- AchievementColorRuleSet: ordered conditions, thresholds, semantic colors, version.
- RotationPlaylist: name, enabled state, ordered dashboard entries, per-item duration, default duration.
- AssetReference: approved storage key/URL, type, size/dimensions, creator metadata.
- AuditEvent: actor, action, target, timestamp, safe change summary.

Choose persistence only after reviewing the existing app, authentication, and deployment. Do not add these tables without owner approval.

## 10. Security, accessibility, and performance

- Keep credentials/secrets server-side; use least-privilege PostgreSQL accounts and read-only access for metric queries where possible.
- Enforce authorization server-side; validate and allowlist filters and identifiers; parameterize queries.
- Configure restrictive CORS, security headers, rate limits, request timeouts, safe errors, and redacted logs.
- Validate image uploads and avoid unsafe external asset URLs, path traversal, and arbitrary HTML.
- Target WCAG 2.2 AA: keyboard access, visible focus, semantic labels, chart data alternatives, reduced motion, contrast, and non-color status cues.
- Show loading, empty, error, and stale-data states without presenting loading placeholders as real zero values.
- Resize charts on sidebar collapse, viewport change, theme change, and dashboard rotation.
- Aim for LCP <=2.5 seconds on a representative production connection; establish API latency targets after schema/data volume and hosting are known.
- Fetch active dashboard data first; prefetch only the next rotation item when safe.

## 11. Acceptance criteria

### First dashboard
- Date filters update all related widgets and reject invalid ranges.
- Role totals and IN/OUT percentages match owner-approved examples.
- Sell IN/Sell Out achievement matches approved formulas, including missing and zero targets.
- Working-day progress matches the approved calendar and is independent of sales achievement.
- Threshold boundary tests cover below warning, exactly warning, 99%, 100%, above 100%, and working-day progress boundaries.
- Region navigation works by mouse and keyboard.
- Loading, empty, error, and freshness states are understandable.

### Editor and themes
- View mode prevents accidental layout changes.
- Authorized editor can add/remove/move/resize widgets, edit text, configure background, preview, save, cancel, and reload.
- Invalid layout/settings are rejected safely.
- Light/dark/system themes are readable and persist as intended.
- Chrome and Firefox layouts fit after sidebar collapse and at supported viewport sizes.

### Rotation
- Admin can choose dashboards/order and duration; default is 10 seconds.
- Pause/resume/next/previous/exit work without duplicate timers.
- Rotation stops or is blocked during editing/unsaved changes.
- Failed dashboard behavior is visible and tested.

### Engineering quality
- CI passes lint, typecheck, unit tests, build, and required E2E checks.
- No secrets, guessed production schema, or hardcoded production metrics are committed.
- Setup, environment variables, metric definitions, API contract, and deployment notes are documented.

## 12. Incremental roadmap

### Phase 0 — Discovery and data contract
Inspect repository structure, package manager, current UI/API, tests, database access, authentication, deployment, and CI. Owner provides schema, data dictionary, representative non-sensitive examples, metric formulas, and timezone/calendar rules. Agree on auth, hosting, asset storage, and config persistence.

**Exit:** repository assessment and approved metric/API contracts; no guessed schema.

### Phase 1 — Foundation
Adapt TypeScript/build/lint/test setup; implement responsive app shell, collapsible sidebar, routing, light/dark/system theme, error handling, and health check. Use clearly marked mock fixtures.

**Exit:** shell works in Chrome/Firefox and CI passes.

### Phase 2 — First read-only Sales Performance dashboard
Implement filters, team totals, KPI cards, region navigation, Sell IN/Sell Out chart, and working-day progress. Connect PostgreSQL only after schema/formulas are approved.

**Exit:** dashboard matches validated business outputs.

### Phase 3 — Configurable achievement rules
Build admin settings, rule precedence, threshold/color editing, preview, validation, persistence, and boundary tests.

### Phase 4 — Dashboard editor
Build typed widget registry, drag/drop/resize, add/remove/configure, text blocks, background image settings, preview/save/revert, and layout versioning.

### Phase 5 — Playlist and presentation mode
Build admin playlist, ordering, per-dashboard duration, 10-second default, rotation controller, kiosk mode, and playback controls.

### Phase 6 — Additional sidebar dashboards
Add each dashboard separately when its data schema, KPI definitions, and acceptance examples are supplied. Reuse common filters, design tokens, widgets, API contracts, and tests.

### Phase 7 — Production hardening
Security review, accessibility audit, performance/load tests, observability, backup/restore, deployment runbook, and user acceptance testing.

## 13. Product-owner decisions needed

1. Provide PostgreSQL schema/table definitions for sales facts, targets, regions, roles/people, and working days.
2. Clarify whether DIST and SE are distinct and which belongs in KPI cards.
3. Confirm achievement formulas, aggregation order, and zero/missing target handling.
4. Confirm IN/OUT meaning and whether counts are people with achievement >=100%.
5. ~~Approve conditional-color precedence.~~ **Resolved (owner-confirmed):** see section 17.8.
6. Confirm business timezone and working-day semantics.
7. Confirm identity provider and Viewer/Editor/Admin permissions.
8. Confirm hosting and approved image storage.
9. Decide whether layouts/playlists are global, per role, per user, or per display.
10. Decide whether filters persist during rotation or each dashboard retains separate filter state.

## 14. Instructions for the coding agent

- Inspect the whole repository before implementation and report architecture, package manager, tests, DB access, and deployment assumptions.
- Treat this document as requirements, not proof that a framework or schema already exists.
- Preserve sound existing patterns; do not rewrite everything or install the whole stack in one change.
- Do not create/alter business tables or guess column names until the owner supplies the schema.
- Use mock contracts first where schema is unknown, clearly marked as mock.
- Keep metric and threshold logic in tested domain/API functions, not duplicated in UI.
- Do not hardcode regions, headcounts, playlist, or thresholds in presentation components.
- Never commit secrets or real customer/sales data.
- Work in small reviewable PRs. Run available lint, typecheck, unit, build, and E2E checks and report anything not run.
- Do not merge or deploy without explicit authorization.


---

# Detailed Specification Addendum

This addendum extends the PRD into a shared product and engineering contract. It is intended to be readable by product owners, analysts, designers, developers, QA engineers, and AI coding/review agents. If an implementation detail conflicts with an unapproved assumption here, preserve existing behavior and raise the decision rather than silently inventing a business rule.

## 15. Requirement authority, evidence, and decision status

Every requirement must be traceable to one of these categories:

- **Owner-confirmed:** explicitly confirmed by the product owner or supported by an approved data dictionary/example.
- **Reference-observed:** visible in the supplied Tableau screenshot, but not necessarily a complete business specification.
- **Proposed default:** a recommendation awaiting owner approval.
- **Open decision:** cannot be finalized until the owner supplies data, policy, or operational constraints.
- **Implementation detail:** engineering choice that may be adjusted if the repository has a sound established pattern.

The reference screenshot is evidence of the desired visual arrangement, not evidence of database schema, exact metric formula, all possible states, or responsive behavior. Every production metric must be tied to an approved definition and an example that can be independently recalculated.

Maintain a decision log in the PR or project documentation when a material assumption is approved. Do not treat mock data as business evidence.

## 16. End-to-end data contract and lineage

Every dashboard item should be traceable through the following chain:

1. **Source:** approved PostgreSQL table/view and relevant columns, or an explicitly approved API source.
2. **Grain:** what one source row represents (for example, transaction line, sales person per day, region per period, or working-calendar date).
3. **Scope:** date boundaries, selected region, role, active status, business unit, and any security restrictions.
4. **Transformation:** joins, deduplication, exclusions, aggregation, target matching, and null/zero handling.
5. **Metric:** formula, unit, precision, denominator, and rounding policy.
6. **API DTO:** typed, validated output including as-of/freshness metadata where relevant.
7. **UI binding:** widget identifier and the exact DTO field(s) it renders.
8. **Verification:** a test fixture or owner-approved hand-calculated example that confirms the result.

For each metric, document the SQL/query location, query grain, filter behavior, and a test that detects double counting. Never aggregate facts from different grains before joining unless the aggregation strategy is explicitly safe. Do not use a client-side calculation for authoritative financial/sales values when the server can calculate them consistently.

### Data-quality policy

- Distinguish zero from unknown/null. A missing target must not silently become zero or 100%.
- Define treatment for duplicate facts, inactive employees, missing region mappings, returns/negative sales, late-arriving records, and records updated after the first query.
- Define whether date filters use business-local calendar dates and whether the end date is inclusive. Recommended API convention: date-only inputs in the business timezone, inclusive start and end dates, translated into a safe half-open timestamp range internally when source columns are timestamps.
- Keep display rounding separate from calculation precision. Compare unrounded values for thresholds; round only for presentation unless the owner explicitly defines a different rule.
- Include a freshness/as-of timestamp and a source/data warning when a result is stale or incomplete.
- Use stable IDs for regions, roles, dashboards, and widgets; display labels may change without breaking saved configuration.

## 17. Dashboard inventory and item-by-item calculation contract

The inventory below covers the supplied screenshot. The formulas are provisional where source schema or business rules have not been supplied. Implementation must use domain/service functions and server query results rather than embedding formulas in chart components.

### 17.1 Start Date and End Date filters

**Purpose:** Define the reporting period used by all date-scoped metrics.

**Inputs:** start date, end date, optional dashboard-specific filters.

**Rules:**
- Validate date format and reject start date later than end date.
- Confirm default date selection with the owner; do not assume the current month is final policy.
- Apply one canonical filter object to all widgets whose scope includes the global date filter.
- Show the active period visibly and provide reset-to-default.
- Preserve the user's filter state according to the agreed navigation/rotation policy.
- Prevent stale asynchronous responses from overwriting the latest selection.

**Data flow:** browser filter state -> validated API query -> authorization and date normalization -> repository queries -> typed response -> query cache -> widgets.

**Acceptance:** changing either date updates every applicable KPI and chart consistently; invalid ranges issue no data query; date boundaries match owner-approved examples.

### 17.2 Team totals: RSM, ASM, SPV, SE

**Purpose:** Show the total number of people/entities in each role for the selected scope.

**Provisional formula:**
- `role_total(role, scope) = COUNT(DISTINCT approved_person_key)` for people assigned to that role and matching approved active/effective-date rules.

**Must be decided:** authoritative person key, role assignment history, active/inactive rules, multi-role users, date-effective logic, region scope, and whether counts reflect current roster or roster as of the reporting period.

The header's RSM/ASM/SPV/SE labels must be mapped from authoritative role definitions. Do not infer that DIST and SE are synonyms.

**Acceptance:** each total matches an owner-approved source query and remains stable when a label changes.

### 17.3 Role cards: RSM, ASM, SPV, DIST (or owner-confirmed categories)

Each card has a role total and separate IN and OUT status summaries.

**Owner-provided example:** RSM (5) means five RSMs. IN 20% / 1 means one of those five has the relevant achievement >=100%; therefore 1 / 5 * 100 = 20%.

**Provisional formulas:**
- `role_headcount = count of distinct eligible people for the role in scope`
- `in_achieved_count = count of eligible people whose approved IN achievement >= 100%`
- `in_achieved_pct = in_achieved_count / role_headcount * 100`
- `out_achieved_count = count of eligible people whose approved OUT achievement >= 100%`
- `out_achieved_pct = out_achieved_count / role_headcount * 100`

**Edge cases:** if headcount is zero, show “—” or “No members” instead of NaN/Infinity. If a person's achievement is unknown due to missing target/data, do not silently count them as achieved or failed; display data-quality context and apply the owner-approved denominator policy. Calculate counts from person-level achievement values, not from an average role achievement, unless the owner defines another rule.

**Open decisions:** confirm whether IN is Sell IN and OUT is Sell Out; confirm DIST/SE; confirm whether people with no transactions remain in the denominator; confirm whether achievement exactly 100% qualifies (the example says >=100%).

**Acceptance:** test count and percentage for 0/0, 0/5, 1/5, 5/5, missing target, and achievement exactly 100%.

### 17.4 Sell IN achievement bar

**Purpose:** Compare Sell IN performance with its target for the selected region and period.

**Provisional formula:**
- `sell_in_achievement_pct = approved_aggregated_sell_in_actual / approved_aggregated_sell_in_target * 100`

The aggregation method must be specified. Normally, aggregate actual and target at the intended scope first, then divide; do not average row-level percentages unless explicitly required.

**Displayed value:** percentage, formatted using the approved precision (reference image displays whole percentages on bars).

**Data needed:** source grain, actual measure, target measure, period alignment, region mapping, exclusions, target changes over time, and zero/missing target policy.

**Acceptance:** the displayed value equals the independently calculated approved example; tooltips expose actual, target, and period when those values are approved for display.

### 17.5 Sell Out achievement bar

Same contract as Sell IN, but using the owner-approved Sell Out actual and target definitions. Do not assume Sell Out is identical to shipment, invoice, or sell-through without confirmation.

**Provisional formula:**
- `sell_out_achievement_pct = approved_aggregated_sell_out_actual / approved_aggregated_sell_out_target * 100`

Test zero target, missing target, negative/adjusted facts where applicable, and values above 100%.

### 17.6 Region selector / region panels

**Purpose:** Partition chart results by the approved regional hierarchy.

- Region IDs, display labels, and ordering come from an approved data source or configuration.
- Selecting a region must scope both Sell IN and Sell Out series consistently.
- If the chart displays multiple regions at once, the same filter contract applies; do not combine regional and all-region aggregates in a way that double-counts totals.
- Handle deleted/renamed regions and no-data regions gracefully.
- Keyboard focus and selected state must be visible and programmatically announced.

**Acceptance:** selected region and API response agree; switching regions cannot leave one series from the previous selection.

### 17.7 Working-day progress circular indicator

**Purpose:** Show progress through the approved working calendar, used as a benchmark for expected achievement.

**Owner-confirmed constraint:** this percentage comes from the working-day table/calendar, not sales achievement. The screenshot example is 66.67%.

**Provisional formula:**
- `working_day_progress_pct = elapsed_working_days / total_working_days_in_period * 100`

This formula is not complete until the owner defines whether elapsed days are counted through today, how the selected period interacts with today, and how holidays/non-working days are represented. The server/API is authoritative.

**Required policy decisions:** business timezone, current/as-of date, inclusive boundaries, holidays, weekends, partial periods, future periods, periods with no working days. Progress must be capped to 0-100% (owner-confirmed requirement for the color rule in 17.8).

**UI:** circular indicator, percentage, clear “Working-day progress” label, as-of date, and explanatory tooltip. It must not be described as achievement progress.

**Acceptance:** tests include a period with 3 elapsed working days out of 5 (60%), weekends/holidays, first/last period day, future dates, and zero working days once owner-approved semantics are defined.

### 17.8 Conditional bar colors

**Status:** Owner-confirmed.

**Purpose:** Visually compare achievement against elapsed working-day progress and a configurable warning band.

**Inputs:** `achievement` (unrounded %), `workingDayProgress` (unrounded %, server-capped to 0-100), `warningMin` (default 70), `successMin` (default 100).

**Rule (evaluated in this order; first match wins):**
1. If `achievement >= workingDayProgress` -> **green / on-track**. Equality is green. Applies even when achievement is below `warningMin`.
2. Else if `achievement < warningMin` -> **red / behind**.
3. Else if `achievement < successMin` -> **yellow / warning**.
4. Else -> **green / on-track** (safe fallback; unreachable when working-day progress <= 100).

Condition 1 can only yield green. When it does not apply, the result comes from the achievement band (conditions 2-4). The working-day comparison alone never produces a final red.

```
status(ach, wd):
  if ach is null:  return unavailable
  if wd is not null and ach >= wd:  return green
  if ach < warningMin:  return red
  if ach < successMin:  return yellow
  return green
```

If working-day progress is missing, skip step 1, evaluate the achievement band only, and show a data-quality warning.

**Reference test table (working-day progress = WD, achievement = ACH):**

| WD | ACH | Result | Reason |
|---|---|---|---|
| 80% | 75% | Yellow | ACH < WD, falls through; 70% <= ACH < 100% |
| 80% | 60% | Red | ACH < WD and ACH < 70% |
| 66.67% | 66.67% | Green | ACH = WD counts as on-track |
| 66.67% | 66.66% | Red | ACH < WD and ACH < 70% |
| 90% | 70% | Yellow | Exactly 70% is in the yellow band |
| 90% | 69.99% | Red | Just below warning minimum |
| 90% | 99.99% | Yellow | Not yet 100% |
| 90% | 100% | Green | Fallback / band green |
| 40% | 50% | Green | ACH >= WD, even though ACH < 70% |
| 100% | 100% | Green | ACH >= WD |
| missing | 75% | Yellow | Skip step 1, band only, show warning |
| any | missing | Unavailable (neutral) | Never colored red or green |

Compare unrounded values; round only the displayed label. Status must also be exposed as text/icon, not color alone.

**Settings contract:** store rule order, threshold(s), colors, inclusive/exclusive boundaries, and fallback behavior in validated configuration. Use semantic tokens (success/warning/danger) mapped to accessible theme colors; allow administrators to configure supported colors while enforcing contrast and preserving non-color status labels.

**Comparison precision:** compare unrounded numeric values; round only the displayed label.

**Acceptance boundary tests:** every row of the reference test table above; achievement just below/equal/above working-day progress; 69.99%, 70%, 99.99%, 100%; missing achievement; missing working-day progress; invalid threshold configuration.

### 17.9 Display date and refresh metadata

- Use an explicit configured business timezone for the business display date; do not rely on the browser's local timezone if they differ.
- Display data as-of/last refreshed separately from the current display date.
- If data is stale, show a non-blocking freshness indicator and preserve the last known good data only with clear stale status.
- The refresh interval and stale threshold remain operational decisions based on source update cadence and query load.

### 17.10 Text and image widgets

These are user-authored content rather than calculated metrics.
- Text blocks store safe structured text/limited formatting, widget position, style tokens, and optional links subject to URL policy.
- Image/background widgets store an approved asset reference, fit mode, position, optional overlay, and accessibility text where meaningful.
- Do not store executable markup or allow script execution. Decorative backgrounds need appropriate contrast overlays and should not be announced as meaningful content to screen readers.
- Provide fallback when an asset is missing or inaccessible.

## 18. Design system and detailed UI/UX specification

### 18.1 Design principles

- **Professional analytics first:** legible data, clear hierarchy, minimal decoration, predictable interactions.
- **Dense but not cramped:** preserve the information density of the Tableau view while improving spacing and readability.
- **Configurable, not arbitrary:** allow layout flexibility within a stable design system and supported widget capabilities.
- **Consistent semantics:** the same metric, status, filter, and action should look and behave the same across dashboards.
- **Responsive by default:** adapt to available width/height rather than treating one screenshot resolution as the layout specification.
- **Accessible by design:** keyboard operation, semantic labels, contrast, chart alternatives, and reduced-motion support.

### 18.2 Recommended frontend/UI stack

Use the stack from section 7, with these implementation boundaries:

- React + Vite + TypeScript for UI and build.
- Tailwind CSS for utilities and design tokens.
- shadcn/ui + Radix primitives for buttons, tabs, dropdowns, dialogs, tooltips, forms, and accessible interaction patterns.
- Lucide React for icons; avoid mixing icon libraries without a reason.
- Apache ECharts + echarts-for-react for charting; use one charting system initially to keep interaction and theme behavior consistent.
- react-grid-layout behind a local adapter for dashboard editor positioning/resizing. Verify React and package version compatibility before adoption.
- TanStack Query for server data; Zustand only for cross-component client UI/editor/playback state.
- React Hook Form + Zod for configuration forms and runtime validation.
- date-fns for presentation/date utilities; server owns working-day calculations.
- Vitest + Testing Library + jest-axe and Playwright for tests.

Do not add Redux, a second charting library, multiple overlapping form libraries, or a second ORM unless a documented need justifies it.

### 18.3 Layout zones

The standard dashboard shell should have:
1. **Sidebar:** collapsible navigation, dashboard groups, current selection, and accessible expand/collapse control.
2. **Top bar:** page title/breadcrumb, global filters where relevant, theme control, refresh/freshness indicator, and authorized actions.
3. **Dashboard canvas:** responsive grid containing cards, charts, text, and other registered widgets.
4. **Edit toolbar (edit mode only):** add-widget action, undo/cancel where supported, save/publish, and unsaved-change status.
5. **Presentation controls (presentation mode):** unobtrusive pause/resume, next/previous, exit, and optional remaining-time indication.
6. **Admin settings:** navigation for achievement rules, playlists, layout permissions, and approved asset configuration.

Do not force every dashboard to use every zone; keep the shell consistent while allowing dashboard-specific content.

### 18.4 Visual tokens and typography

Define theme tokens centrally rather than hardcoding colors per component:
- Canvas/background, surface, elevated surface, border/divider, primary text, secondary text, muted text, focus ring.
- Brand/accent, success, warning, danger, informational status.
- Chart series colors, gridlines, tooltip surfaces, selected-tab states, and focus/hover states.
- Spacing scale, border radius, shadows/elevation, font families, font sizes, line heights, and chart label sizes.

Use a neutral professional base with a restrained accent color. The screenshot's orange/blue treatment may inform an optional brand preset, but should not dictate all components or reduce contrast. Light and dark themes must use semantic tokens and remain visually coherent.

### 18.5 Responsive behavior and sizing

- Use CSS grid/flex layouts and responsive breakpoints. Avoid fixed viewport-specific pixel placement for the full dashboard.
- Use container-aware chart resizing; trigger resize after sidebar state changes, grid changes, theme changes, and viewport changes.
- On narrow widths, stack or reflow cards and chart regions; do not shrink labels to unreadable sizes.
- If a data table requires horizontal scrolling, constrain scrolling to the table region rather than the entire page.
- Define and test a minimum supported viewport during Phase 1 after inspecting target display devices.
- Support browser zoom and high-DPI screens.
- Rotation must not change dimensions in a way that causes visible layout jumps between dashboards with different widget counts.

### 18.6 Interaction states

Every interactive component must define:
- Default, hover, focus-visible, pressed/selected, disabled, loading, empty, error, and success states where applicable.
- Keyboard navigation and focus restoration after dialogs close.
- Clear confirmation for destructive actions such as removing a widget or deleting a playlist.
- Undo or cancel for editing flows where feasible.
- Toast/inline feedback that does not steal focus or disappear before it can be read.
- Confirmation or a draft state before publishing shared configuration.

### 18.7 Chart UX

- Tooltips should identify series, region, period, actual/target where available, and formatted achievement.
- Axis units, number formatting, labels, and legends must be consistent.
- Do not rely exclusively on red/green color; include text, symbols, or accessible descriptions.
- Prevent clipped labels and overlapping bars on small containers.
- Provide a table or textual summary for screen readers and users who cannot interpret the chart.
- Respect reduced-motion preferences and avoid animated transitions during rapid playlist rotation.
- Chart colors must respond to light/dark theme and configured rules.

## 19. Frontend component and domain boundaries

Suggested component responsibilities (adapt to the actual repository):

- `AppShell`: responsive layout, sidebar, top bar, route outlet.
- `DashboardRoute`: loads dashboard metadata and handles route-level states.
- `GlobalFilterBar`: date filters and shared filter state.
- `DashboardGrid`: renders layout from validated widget definitions.
- `WidgetRenderer`: resolves an allowlisted widget type to a component.
- `KpiCard`: presentational display only; receives calculated values.
- `AchievementBarChart`: presentational chart; receives values and evaluated semantic statuses.
- `WorkingDayProgress`: presents authoritative API progress and metadata.
- `DashboardEditor`: drag/resize/add/remove and layout draft state.
- `ThemeProvider`: light/dark/system theme tokens and persistence.
- `RotationController`: playlist state machine and timing, independent of chart components.
- `AdminSettings`: validated forms for rule and playlist settings.

Domain modules should separately own:
- Metric DTOs and formatting contracts.
- Achievement formula/aggregation policy.
- Role achievement count logic.
- Working-day progress semantics.
- Threshold evaluation and precedence.
- Layout schema/version migration.
- Playlist validation and playback transitions.

Presentation components must not query PostgreSQL, contain SQL, infer business formulas, or own independent rotation timers.

## 20. Backend/API contract expectations

Use modular route/service/repository boundaries:
- Route layer: authentication/authorization, input parsing, response mapping.
- Service/domain layer: business rules, calculations, validation of state transitions.
- Repository layer: parameterized PostgreSQL access and query-specific aggregation.
- Contract layer: request/response DTOs, Zod schemas, stable error shape.
- Configuration layer: environment variables and safe runtime defaults.

Each endpoint must document purpose, permissions, filters, response fields, error cases, cache policy, and freshness. Proposed endpoints in section 8 remain provisional until repository inspection.

Suggested response metadata for metric responses:
- `dataAsOf`: source data timestamp when available.
- `generatedAt`: API response generation timestamp.
- `period`: normalized start/end date and timezone.
- `filtersApplied`: safe normalized filter summary.
- `warnings`: machine-readable non-sensitive data-quality warnings.
- `data`: typed metric payload.

Do not return SQL, connection strings, stack traces, or internal sensitive identifiers to the browser. Validate all configuration on both read and write; malformed persisted configuration should fall back safely and produce an operational warning.

## 21. Dashboard configuration and persistence lifecycle

Treat dashboard layout, business facts, and runtime UI state as distinct categories:

- **Business facts:** remain in the existing PostgreSQL schema; read through approved repositories.
- **Dashboard definitions/layouts:** app configuration with schema version and owner/editor metadata.
- **Achievement rules:** versioned admin configuration.
- **Playlists:** ordered persistent dashboard references and durations.
- **Assets:** references to approved object/file storage.
- **Transient UI state:** open sidebar, current tab, unsaved drag position, and temporary playback state.

Before adding persistence tables, inspect current configuration mechanisms and request owner approval. If tables are approved:
- Include explicit migrations, constraints, indexes, and rollback guidance.
- Separate app-owned configuration tables from source business tables.
- Keep revision history for shared layouts/rules where practical.
- Validate referenced dashboards/widgets exist before publishing a playlist/layout.
- Define concurrency behavior for two editors saving the same layout (optimistic version check recommended).
- Never silently overwrite a newer revision.

## 22. Error, fallback, and recovery scenarios

Document and test behavior for:
- PostgreSQL/API unavailable: show actionable error and retry; never show zeros as if valid.
- Empty result set: explain no data matches the filters.
- Partial data or missing targets: show “unavailable” and a data-quality warning per approved policy.
- Invalid or obsolete layout version: use a safe default layout and report a recoverable configuration error.
- Missing background asset: use theme surface fallback.
- Invalid threshold rules: reject save and keep the last valid published configuration.
- Dashboard fails during rotation: retain last good view or skip to next according to an approved setting; show a non-blocking indicator.
- Session/permission expiry: preserve safe draft state if feasible, require reauthentication, and reject unauthorized writes.
- Two editors conflict: prompt to reload/compare rather than silently losing changes.
- Browser resize or sidebar toggle: charts reflow without requiring a page reload.

## 23. Testing strategy and traceability

For each requirement, add a stable identifier and link implementation/tests to it where practical. Suggested prefixes:
- `DATA-`: data contract and source lineage.
- `METRIC-`: formulas and aggregation.
- `UI-`: shell, filters, themes, responsiveness.
- `EDIT-`: layout editor and widget registry.
- `RULE-`: conditional colors/settings.
- `PLAY-`: playlists/rotation.
- `SEC-`: security/permissions.
- `A11Y-`: accessibility.
- `OPS-`: reliability and deployment.

Required testing layers:
1. **Pure unit tests:** formulas, threshold precedence, date normalization, layout validation, playlist transitions.
2. **Repository/query tests:** approved SQL against a test database or controlled fixtures; validate aggregation grain and filters.
3. **Component tests:** filters, KPI states, chart fallback, editor controls, settings forms.
4. **API tests:** schema validation, permissions, errors, and response contracts.
5. **Browser E2E:** date filters, region switching, theme switching, sidebar resizing, editing/save/reload, playlist timing and controls.
6. **Accessibility checks:** automated axe checks plus manual keyboard and screen-reader review.
7. **Visual regression (optional):** use stable fixtures for screenshot comparison after visual design is agreed.

Do not assert exact real metrics from mock fixtures as production truth. Each owner-approved metric example should include inputs, expected intermediate values, final output, and source/query provenance.

## 24. Definition of ready for implementation

A dashboard metric is ready for production implementation when:
- Source table/view and column definitions are supplied.
- Grain and join keys are documented.
- Formula, denominator, date rules, rounding, and null/zero behavior are approved.
- At least one independently verified example is available.
- Authorization and filter scope are understood.
- API DTO and UI display/empty/error behavior are defined.
- Tests can assert expected results without relying on production customer data.

The dashboard shell, theme, navigation, layout editor scaffolding, and mock-backed UI can progress before all business schema is known, provided mock data is clearly labeled and no irreversible data-model assumptions are introduced.

## 25. Agent/human handoff checklist

Before coding:
- Read this PRD and any decision log.
- Inspect current source, tests, migrations, CI, and deployment.
- Identify which requirements are owner-confirmed, proposed, or open.
- State assumptions and ask only the questions that block the next milestone.

During coding:
- Make small commits and keep code aligned with module boundaries.
- Update the PRD or decision log when a requirement changes.
- Add tests with each business rule or interactive behavior.
- Keep schema assumptions out of production paths.
- Avoid broad dependency changes without compatibility review.

Before opening a PR:
- Summarize requirements implemented and IDs addressed.
- Explain changed files and architecture decisions.
- List tests run and their actual results.
- Document unresolved decisions, limitations, and any migration/rollback impact.
- Confirm no credentials or real customer data are included.

A coding agent must not claim completion based solely on code generation. Completion requires observable behavior, passing checks, and evidence linked to the acceptance criteria.
