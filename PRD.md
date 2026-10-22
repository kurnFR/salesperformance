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
- Recommended default precedence:
  1. Achievement >=100%: success color.
  2. Otherwise achievement >= working-day progress: on-track color (green).
  3. Otherwise achievement >= configurable warning minimum (default 70%): warning color (yellow).
  4. Otherwise: behind-target color (red).
- This precedence resolves the overlap between the example “above working-day progress is green” and “70%–99% is yellow.” The owner must approve it before release.
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
5. Approve conditional-color precedence.
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
