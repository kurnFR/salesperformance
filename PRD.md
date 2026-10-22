# Sales Performance Dashboard — Product Requirements Document

**Status:** Draft v2 (living specification)  
**Repository:** https://github.com/kurnFR/salesperformance  
**Product:** Interactive web replacement for the existing Tableau Sales Performance dashboard

This file is the short entry point: what we are building, why, for whom, and how the rest of the specification is organised. Detail lives in the documents listed in [Document map](#document-map). The previous single-file PRD is preserved in git history (commit `15f4b3b`).

## 1. Summary

Build a maintainable, responsive BI web application backed by the existing PostgreSQL database. Reproduce the supplied Tableau dashboard as the first view, then add configurable layouts, filters, light/dark themes, dashboard playlists and administration incrementally. It must work in current Chrome and Firefox and adapt to viewport changes.

The owner will supply database tables and business definitions later. Until then: do not guess the production schema, do not create business tables, do not fabricate production metrics. Use typed contracts and clearly marked mock fixtures.

## 2. Problem and success measures

**Problem (proposed wording, confirm):** The Tableau dashboard is hard to customise, cannot be extended one dashboard at a time with consistent rules, and its colour/threshold logic is not configurable by admins.

**Success measures** (all `[P]` proposed, owner to confirm targets):

| Measure | Target |
|---|---|
| Metric parity with Tableau on agreed sample periods | 100% match after approved rounding |
| Owner-approved hand-calculated examples passing as automated tests | 100% |
| Time for an admin to change thresholds/colors | No code change or deploy |
| Critical/serious accessibility violations (axe) | 0 |
| First dashboard LCP on a representative connection | <= 2.5 s |

## 3. Goals and non-goals

**Goals**
- Replace the Tableau dashboard using the existing PostgreSQL database as the source of truth.
- Match the reference: Start/End Date filters, role KPI cards, region navigation, Sell IN / Sell Out achievement bars, working-day progress, display date.
- Keep metric definitions consistent, server-calculated and testable.
- Let authorised editors add/remove widgets, drag and resize, edit text, and set backgrounds.
- Let admins configure achievement thresholds/colors without code changes.
- Provide an admin-managed playlist that rotates dashboards automatically (default 10 s each).
- Build each sidebar dashboard incrementally on shared components and a shared design system.

**Non-goals for the first release**
- Replacing PostgreSQL or changing its business schema.
- Guessing joins, column names, KPI formulas or role-label meanings.
- A general-purpose arbitrary SQL editor.
- Building every future dashboard in the first milestone.
- Export (CSV/PNG/PDF), drill-through, annotations and alerting (candidates for later phases; confirm).

## 4. Users and permissions

| Role | Description |
|---|---|
| Viewer | View dashboards, use permitted filters, inspect values, use presentation mode |
| Display (kiosk) `[P]` | Unattended screen showing a playlist; read-only; long-lived display credential |
| Editor | Viewer + add/remove/move/resize widgets, edit text/appearance, preview, save layouts |
| Administrator | Editor + manage dashboards, achievement rules, backgrounds, playlists, order, durations, access |
| Technical administrator | Secrets, deployment, PostgreSQL connectivity, monitoring (no product UI rights implied) |

**Permission matrix** (`[P]`; enforced on the server, never only by hiding UI):

| Capability | Viewer | Display | Editor | Admin | Tech admin |
|---|:-:|:-:|:-:|:-:|:-:|
| View dashboards / change permitted filters | Yes | Yes | Yes | Yes | — |
| Presentation mode / playlist playback | Yes | Yes | Yes | Yes | — |
| Edit layouts, text, widgets | — | — | Yes | Yes | — |
| Achievement rules, playlists, assets, access | — | — | — | Yes | — |
| Secrets, DB connection, deployment | — | — | — | — | Yes |

Row-level data scope (for example, an RSM seeing only their own region) is an open decision: see `docs/decisions.md` (DEC-011).

## 5. Requirement authority

Every requirement carries a tag:

- `[C]` **Owner-confirmed** (explicit confirmation or approved data dictionary/example)
- `[R]` **Reference-observed** (visible in the Tableau screenshot; not a full business spec)
- `[P]` **Proposed default** (recommendation awaiting owner approval)
- `[O]` **Open decision** (needs owner data or policy)
- `[I]` **Implementation detail** (engineering choice; adjustable)

The screenshot is evidence of visual arrangement, not of schema, formulas or responsive behaviour. Mock data is never business evidence. Requirements have stable IDs with these prefixes: `DATA-`, `METRIC-`, `RULE-`, `UI-`, `EDIT-`, `PLAY-`, `SEC-`, `A11Y-`, `OPS-`.

## 6. Roadmap at a glance

| Phase | Outcome | Needs from owner |
|---|---|---|
| 0 | Discovery, approved data contract and decisions | Schema, data dictionary, examples, calendar rules |
| 1 | App shell, routing, themes, health check, mock fixtures | — |
| 2 | First read-only Sales Performance dashboard on real data | Approved formulas + verified examples |
| 3 | Configurable achievement rules | Rule defaults confirmed |
| 4 | Dashboard editor | Persistence approval |
| 5 | Playlist and presentation mode | Kiosk auth decision |
| 6 | More sidebar dashboards, one at a time | Per-dashboard schema + KPIs |
| 7 | Production hardening | — |

Details, exit criteria and acceptance tests: `docs/05-roadmap-and-testing.md`.

## Document map

| File | Contents |
|---|---|
| `PRD.md` | This overview |
| `docs/01-metrics-contract.md` | Metric definitions, lineage, data-quality policy, color rule, per-item calculation contract |
| `docs/02-architecture.md` | Stack, module boundaries, API contract, flow diagrams (Mermaid) |
| `docs/03-data-model.md` | App-owned data structures (JSON), proposed config tables, source-mapping template |
| `docs/04-ui-ux-spec.md` | Functional UI requirements, design system, responsive, interaction, chart UX, accessibility |
| `docs/05-roadmap-and-testing.md` | Phases, acceptance criteria, error scenarios, testing strategy, definition of ready |
| `docs/decisions.md` | Decision log with proposed defaults and status |
| `docs/reference/` | Reference screenshot and Tableau artifacts |
| `AGENTS.md` | Rules and workflow for AI coding agents |


## 7. Detailed metric and UI contract

### 7.1 Traceability requirement

Every displayed value must be traceable end-to-end: approved source table/view and columns; row grain; joins; date/region/role filters; exclusions; aggregation order; formula; rounding; zero/missing behavior; business timezone and as-of time; API DTO; UI component; accessibility description; and automated test with owner-approved expected output. Give each metric a stable ID and document it in docs/01-metrics-contract.md. No metric is production-ready until its source mapping and example calculation are approved.

### 7.2 Shared calculation rules

- Use decimal/numeric arithmetic for amounts and ratios; round only at the approved display/business stage.
- Calculate ratios from aggregated numerator and denominator unless the owner explicitly specifies another method.
- A zero denominator is undefined/N/A by default, not automatically 0% or 100%.
- Missing actuals, targets, duplicate targets, overlapping role assignments, and incomplete calendars must be represented as data-quality states, not silently converted into plausible numbers.
- Define inclusive/exclusive date boundaries and one business timezone. Recommended default is inclusive business dates translated into a half-open timestamp range for database queries.
- Apply row-level authorization before aggregation to prevent leakage through totals.
- The API is authoritative for business calculations and working-day progress; the browser formats and visualizes normalized results.

### 7.3 Sales Performance dashboard item logic

| Item ID | Item | Input | Calculation/logic | Output |
|---|---|---|---|---|
| UI-FILTER-01 | Start Date | User-selected business date | Validate against End Date and approved reporting range | Updates shared filter state and all dependent widgets |
| UI-FILTER-02 | End Date | User-selected business date | Must be on/after Start Date; boundary convention approved by owner | Updates all dependent widgets; invalid range blocks request |
| UI-HEADER-01 | Dashboard title | Dashboard metadata | No calculation | Configurable title/description |
| UI-HEADER-02 | Display date | Server clock and business timezone | Current business-local date | Date and as-of metadata, not browser-local guess |
| UI-HEADER-03 | Team totals | Approved role/assignment source | Distinct eligible person count per role, using approved active/effective-date rules | RSM, ASM, SPV, SE totals; role labels/order configurable |
| UI-PROGRESS-01 | Working-day progress ring | Approved calendar, selected period, business as-of date | Elapsed eligible working days / total eligible working days × 100 | Percentage, day counts, as-of date and calendar version |
| UI-KPI-01 | RSM card | Eligible RSM population and approved IN/OUT achievements | Total = distinct eligible RSMs. Qualified count = people meeting approved achievement threshold. Qualified % = qualified count / total × 100 | Total, IN %, IN count, OUT %, OUT count |
| UI-KPI-02 | ASM card | ASM population and achievements | Same role-card formula, scoped to ASM | Total, IN %, IN count, OUT %, OUT count |
| UI-KPI-03 | SPV card | SPV population and achievements | Same role-card formula, scoped to SPV | Total, IN %, IN count, OUT %, OUT count |
| UI-KPI-04 | DIST card | Role mapping to be supplied | Do not assume DIST and SE are equivalent; apply same formula only after mapping approval | Configured label and verified values |
| UI-REGION-01 | Region tabs | Approved region dimension and display order | Selection adds a region filter to the chart query | Accessible region navigation and active state |
| UI-CHART-01 | Sell IN bar | Approved Sell IN actual and target | Proposed achievement = aggregated actual / aggregated target × 100 | Bar, percentage, tooltip, actual/target and status |
| UI-CHART-02 | Sell Out bar | Approved Sell Out actual and target | Proposed achievement = aggregated actual / aggregated target × 100 | Bar, percentage, tooltip, actual/target and status |
| UI-STATE-01 | Loading | Request state | Not a business value | Skeleton/status; never show fake zero values |
| UI-STATE-02 | Empty | Valid query with no matching data | No calculation | Explain no matching records and active filters |
| UI-STATE-03 | Error/stale | API/freshness state | Not a business value | Safe error, retry and last-successful-update information |

The supplied image is a visual reference, not authoritative data. Its displayed values must not become production fixtures unless the owner confirms them.

### 7.4 Achievement formula contract

Proposed formula for each measure: achievement percentage = 100 × SUM(eligible actual) / SUM(eligible target). This is not approved until the owner confirms target grain, aggregation order, transaction/posting date, statuses, returns/cancellations, effective dates, and missing/zero-target policy. Ensure joins cannot multiply actuals or targets. Required tests: zero actual, actual equals target, above 100%, zero target, missing target, no facts, duplicate target joins, date boundaries, and multi-day aggregation.

### 7.5 Role attainment contract

The supplied business example says RSM (5) means five RSMs and IN 20% / 1 means one of those five reached achievement >=100%, giving 1 / 5 × 100 = 20%. Proposed formula: population = distinct eligible people assigned to the role under approved effective-date rules; qualified count = distinct people whose approved IN or OUT achievement meets the configured threshold; percentage = qualified count / population × 100. Owner must confirm whether population is current or period-effective headcount, whether people can have multiple roles, and how missing achievement is treated. Missing achievement must not count as achieved by default. A zero-person population returns N/A, not division by zero.

### 7.6 Working-day progress contract

Proposed formula: 100 × elapsed eligible working days / total eligible working days in the selected period. Calendar data must define working/non-working dates and any region-specific calendar or holiday. The elapsed numerator counts eligible working dates within the selected period on or before the approved business as-of date; denominator counts all eligible working dates in the selected period. Owner must decide future periods, historical periods, weekends/holidays as-of behavior, partial current days, region-specific calendars, and zero-working-day periods. Return progressPercent, elapsedWorkingDays, totalWorkingDays, asOfDate, calendarId/version. The example 66.67% must be matched only after the owner supplies the exact period and calendar that yield it.

### 7.7 Conditional achievement color rules

Use one pure, deterministic, unit-tested rule evaluator shared by all charts. Proposed precedence pending owner approval:
1. Missing/invalid achievement or missing progress: neutral/unavailable state with explanation.
2. Achievement >=100%: success green.
3. Otherwise achievement >= working-day progress: on-track green.
4. Otherwise achievement >= configurable warning floor (default 70%): warning yellow.
5. Otherwise: behind-plan red.

This resolves the overlap between “green above working-day progress” and “yellow from 70% to 99%.” Owner approval is required. Admin settings must configure threshold values, semantic colors and priority; validate values and contrast, preview changes, and provide accessible status text. Boundary tests include below 70%, exactly 70%, between 70% and progress, equal to progress, just above progress, 99%, 100%, above 100%, missing data, and progress 0%/100%.

## 8. Professional UI/UX specification

### 8.1 Design direction

Build a restrained enterprise BI interface: high information density without clutter, aligned cards, legible metrics, predictable interactions, clear hierarchy, and consistent light/dark themes. Preserve the reference dashboard's information hierarchy, not necessarily every styling choice. Use neutral surfaces, subtle borders, restrained shadows, consistent radii and spacing, readable sans-serif typography, and tabular numerals for metrics. Use semantic colors only for statuses and ensure contrast in both themes. Include a standard viewing mode and a presentation/wallboard mode.

### 8.2 Exact recommended stack

Resolve compatible stable package versions at implementation time and commit the lockfile. This is the target stack, not an instruction to install everything before repository inspection.

Frontend: Node.js current LTS; React; Vite; TypeScript strict mode; React Router; Tailwind CSS with the Vite-compatible integration; shadcn/ui using only needed Radix UI primitives; lucide-react; TanStack Query for server state; Zustand for small client-only editor/theme/rotation state; React Hook Form and Zod; Apache ECharts and echarts-for-react; react-grid-layout behind a replaceable adapter; date-fns for display/date utilities; and either ky or a shared native-fetch wrapper (choose one).

Backend, if a separate API fits the existing architecture: Fastify; pg; Zod; Pino with redaction; @fastify/helmet; @fastify/cors; @fastify/rate-limit; and optionally @fastify/swagger plus @fastify/swagger-ui. Use Drizzle ORM and drizzle-kit only if app-owned configuration persistence needs them and the repository has no suitable existing data layer. Never create or migrate business tables without approval.

Quality: Vitest; @testing-library/react; @testing-library/user-event; jest-axe; Playwright with Chromium and Firefox; ESLint; typescript-eslint; Prettier. CI should run frozen-lockfile installation, lint, typecheck, unit tests, build, and required E2E tests.

Avoid microservices, duplicate chart libraries, duplicate state managers, arbitrary SQL in layouts, and framework migrations that do not solve a demonstrated problem. If the current repository already uses a sound framework, retain it and implement these boundaries within the existing structure.

### 8.3 Component inventory

Build reusable components for AppShell, CollapsibleSidebar, DashboardHeader, DateRangeFilter, FilterBar, TeamTotals, RoleKpiCard, WorkingDayProgress, RegionTabs, AchievementBarChart, ChartTooltip, AccessibleDataTable, DashboardGrid, WidgetFrame, WidgetPicker, WidgetSettingsPanel, TextWidget, BackgroundSettings, ThemeProvider, PlaylistControls, RotationStatus, AdminSettings, LoadingSkeleton, EmptyState, ErrorState, ConfirmDialog, and UnsavedChangesGuard. Keep UI components presentation-focused; calculations belong in tested domain/API modules.

### 8.4 Responsive and accessible behavior

Desktop uses a collapsible sidebar and flexible main canvas; KPI cards resize/wrap based on available space. Narrow screens use compact/drawer navigation and wrapping filters, with a table alternative for charts. Wallboard mode maximizes chart space while retaining title, as-of time and rotation status. Recalculate charts after sidebar toggle, viewport resize, theme change, widget resize, and dashboard rotation. Target WCAG 2.2 AA: keyboard access, visible focus, semantic labels, reduced motion, contrast, and status cues that do not depend on color alone. Verify Chrome and Firefox via Playwright.

## 9. Input-to-output architecture and traceability

1. Input: approved PostgreSQL source data/calendar plus validated filters.
2. Access boundary: authenticate, authorize dashboard and row scope, validate dates and filter identifiers.
3. Repository: parameterized SQL with explicit grain/joins and duplicate/fan-out prevention.
4. Domain services: aggregate actual/target, compute achievement and role attainment, calculate working-day progress, and classify data quality.
5. API: typed DTO with metric IDs, values, units, status, as-of time, and configuration version.
6. Frontend query layer: fetch/cache/cancel requests and prevent stale responses replacing newer results.
7. Widget registry: render data with validated display settings.
8. Layout engine: apply versioned grid positions and responsive rules.
9. User output: chart/card, labels, tooltip, accessible data summary, freshness and error status.
10. Verification: compare to owner-approved hand calculations and Tableau on agreed periods; preserve fixtures and tests.

Every widget must be traceable by requirement ID to metric ID, API DTO, service, repository query, UI component, configuration, and test. The following are proposed API contracts, not claims that endpoints currently exist: GET /api/health; GET /api/dashboards; GET /api/dashboards/:dashboardId; GET /api/sales-performance/summary; GET /api/sales-performance/regions; GET /api/calendar/working-day-progress; GET/PUT /api/settings/achievement-rules; GET/POST/PUT /api/playlists; POST /api/dashboards/:id/layout. Final paths should follow existing repository conventions.

## 10. Dashboard editor and rotation contracts

The editor has separate View/Edit modes. View mode permits chart interaction but not layout dragging. Layouts use validated, versioned JSON and an allowlisted widget registry; never store executable code or raw SQL. Support add/remove/duplicate/move/resize/reorder, safe text formatting, background image fit/position/overlay, preview, save, cancel/revert, and unsaved-change warnings. Validate image types/sizes and use approved asset storage. Enforce editor/admin permission server-side.

Rotation playlists have an ordered set of dashboards, enabled flags, and per-dashboard durations; default duration is 10 seconds. Provide start/pause/resume/next/previous/exit and presentation mode. Use one rotation controller with timer cleanup, no overlapping timers, and a defined error fallback. Pause or block rotation during edit mode and unsaved changes. Persist playlists through an owner-approved configuration store, not ephemeral browser-only state.

## 11. Data quality, security, and error handling

Metric statuses should distinguish valid, no_data, missing_target, zero_denominator, incomplete_calendar, and error (or an equivalent typed contract). Never represent missing/error data as a genuine zero. Keep credentials server-side; use least-privilege PostgreSQL credentials, parameterized SQL, server-side authorization, restrictive CORS, security headers, rate limits, request timeouts, safe errors, and redacted logs. Sanitize text, validate uploads, and avoid unsafe external asset URLs. Provide retry and last-successful-update information without exposing SQL or stack traces.

## 12. Approval gate and traceability template

Before production metric implementation, owner approval is required for source-to-metric mappings, grain and joins, formulas and aggregation order, role population and IN/OUT semantics, working-day calendar/as-of behavior, color precedence, timezone/date boundaries, missing/zero-target policy, access/row scope, configuration persistence, asset storage, rotation failure behavior, and filter persistence.

For each future widget, record: requirement ID; dashboard/widget; business purpose; owner and approval status; source tables/columns; source grain and joins; filters/exclusions; formula and aggregation order; date boundary/timezone; rounding/format; missing/zero/error behavior; API endpoint/DTO; domain service/metric ID; UI component/configuration; permissions; loading/empty/error behavior; accessibility behavior; approved example input/output; automated tests; and change history/version.
