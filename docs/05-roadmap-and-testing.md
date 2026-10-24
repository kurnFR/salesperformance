# Roadmap, Acceptance and Testing

Related: [`PRD.md`](../PRD.md) · [`decisions.md`](decisions.md)

## 1. Roadmap

Each phase lists tasks and an exit gate. Work in small reviewable PRs, citing requirement IDs.

### Phase 0: Discovery and data contract
- Column names are owner-confirmed (DEC-001). Run the automated schema check (`03-data-model.md` section 4.6) to confirm types, grain, regions and the ETL log, then approve the mapping rows.
- Close blocking decisions in `decisions.md`. For Phase 1 itself only DEC-009, DEC-013, DEC-017 and DEC-008 are needed; Phase 2 additionally needs DEC-077, DEC-028 and DEC-029.
- **Exit:** approved metric and API contracts; no guessed schema.

### Phase 1: Foundation
- Language scaffolding: English and Bahasa Indonesia translation files, admin default language (initial English), per-user override, locale-aware formatting (`I18N-001` to `I18N-007`).
- Responsive test matrix: the four viewports in `04-ui-ux-spec.md` section 10, including half-screen 960 x 1080.
- Database: two pools on one connection (read-only business sessions, app pool for `app_config` only), migrations restricted to `app_config` (`02-architecture.md` section 14).
- Repository tooling (`pnpm` workspaces, TypeScript strict, lint, tests, CI), `.env.example`, column registry, principal registry and the `schema:check` script.
- Monorepo, TypeScript strict, lint, test, CI; `AppShell`, sidebar, routing, theme provider, error boundaries, `/api/health`.
- Repository interfaces with mock implementations; `fixtures/` and golden cases.
- Authentication: admin-created local users (Argon2id, forced change of the temporary password, lockout, admin reset, change password) behind an `AuthProvider` interface. Microsoft sign-in is deferred.
- Department-ready shell: department switcher, department-aware navigation and access model with Sales as the only active department; "Coming soon" pages for planned departments (`10-departments-and-modules.md`).
- Access-control foundation: `AccessContext`, capability catalog, scope-required repositories, `GET /api/me/access`, seed roles as data (`02-architecture.md` section 10). Use a development identity stub until DEC-007 is decided.
- **Exit:** shell works in Chrome and Firefox; CI green; scope tests pass.

### Phase 2: First read-only Sales Performance dashboard
- Filters, team totals, role cards, region navigation, Sell IN/Sell Out chart, working-day indicator, display date, freshness.
- Executive Summary, Regional Manager view and drill-down links (`09-management-experience.md`), reusing these services.
- Connect PostgreSQL only after the relevant mapping rows are `Approved`.
- **Exit:** outputs match owner-validated examples and Tableau parity samples.

### Phase 3: Configurable achievement rules and access admin
- Glossary, default landing per role, saved views (`MGMT-003`, `MGMT-009`, `MGMT-011`).
- Admin screens for roles, capabilities, scopes and user assignments (`UI-019`), with audit.
- Department admin: departments, per-user department switches and department roles (`UI-035`).
- Admin settings, validation, preview, persistence, versioning, boundary tests from the color table.

### Phase 4: Dashboard editor
- Widget registry, drag/resize, add/remove/configure, text blocks, backgrounds, preview/save/revert, versioning, conflict handling.

### Phase 5: Playlist and presentation mode
- Playlist admin, ordering, durations, rotation controller, kiosk mode, playback controls, kiosk authentication.
- Displays (for example the CEO TV), per-display playlists, per-item half/full presentation, the CEO's own playlist management and remote control from a phone (`PLAY-009` to `PLAY-013`), TV mode and on-screen labels (`UI-037`, `UI-038`, `DASH-012`).

### Phase 6: Growth and upload dashboards, then further dashboards
- Order: Sales Detail (pivot) with export and ranking lists, then Growth Sell In, Growth Sell Out (shared matrix component), Daily Upload, then Performance Drivers (`11-insight-dashboard.md`, views 1 to 5 and 7 (Regular and Festive) first, view 6 after the manual-upload sources are labelled, view 8 after the schema check confirms the join and the flag values, DEC-099).
- Rolling 7-month window and growth logic (`GROW-001` to `GROW-008`) get unit tests including year-boundary windows (for example selected Mar-27) and zero last-year values.
- Later dashboards follow the same rule: one at a time with approved schema, KPI definitions and examples.

### Phase 7: AI Chat and Data Findings
- Data Findings: checks, storage, sheet, indicators (`08-data-findings.md`). Can start earlier, after Phase 2, because it only needs source tables.
- AI Chat: tool catalog over the existing services, verifier, pop-up UI, history, evaluation set (`07-ai-chat.md`). Needs DEC-047 (gateway and allowed upstream models) and DEC-046 (data as-of source) first.
- AI provider settings: profiles, encrypted write-only API key, tests, activation, audit (`07-ai-chat.md` section 9).
- **Exit:** golden Q&A parity with dashboards; adversarial and scope tests pass; findings tests pass.

### Phase 8: Production hardening
- Security review, accessibility audit, performance/load tests, observability, backup/restore, deployment runbook, user acceptance testing, Tableau cut-over plan.

## 2. Acceptance criteria

**Sales Detail (pivot)**
- The user can choose period range, dimension headers (with order) and fact columns; a grand total is shown; totals equal server aggregation of the underlying rows, and percent columns equal `SUM(real) / SUM(target)` at every level.
- The owner's example row is reproduced from a mock fixture (Sell IN 2000 and Sell OUT 3000 against target 5000).
- Only allowed principals and regions appear; the grand total is labelled "within your access"; a field id outside the catalog is rejected; the row cap and timeout return a clear message.
- Excel/CSV export equals the on-screen result and carries filters, scope note and as-of.
- **Hierarchy:** expanding a node shows the next level for the same period, filters, scope and measures; a parent equals the sum of its children (percent columns recalculated); facts that cannot be placed on a lower level appear under Unassigned; levels without data show "not available" and never 0; the role's deepest-level cap and the user's scope are enforced at every expansion, including by direct API call; keyboard expand/collapse works.

**Management**
- Executive Summary numbers equal the Sales Performance service output for the same scope; a regional manager never sees another region in tiles, rankings, exports or shared links; derived metrics (`METRIC-019` to `METRIC-021`) match hand calculations including zero remaining days and zero target.

**Daily progress (when a day-level source exists)**
- The line equals cumulative real divided by the monthly target for each day; the pace line equals elapsed working days over total from `dimdate`; colors follow the rule table; missing days are shown as missing; a monthly-only source never offers the view.
- SE-level rows show target, sell-out and achievement; the SE sum versus the distributor's Real Sell OUT difference appears as "Unassigned / difference" and as a data finding.

**Performance Drivers**
- Regular plus Festive equals Total for target and real; mix shares sum to 100%; the product-group selector never mixes groups; a mismatch is shown as a finding, never hidden.
- Shortfall shares sum to 100% and the Pareto line ends at 100%; sell-through and net gap equal hand calculations; streaks use closed months only and break on "n/a"; distribution counts equal the Sales Detail table; YTD equals the sum of months across a year boundary; generated sentences equal their numbers; out-of-scope users see no data, names or counts.

**Resilience (ETL drops, truncates, adds columns)**
- While the source is truncated, locked or missing, dashboards show the last good result with "data update in progress" and never blank or partial numbers; an added column changes nothing; a missing or renamed registered column disables only the affected widget and raises a finding; a missing `app_config` stops the app with restore instructions; the app passes its database tests on PostgreSQL 12 and a current PostgreSQL.

**Window fit**
- Switching the dashboard window between half and full (and any resize) reflows live, with no blank or white area, no scrollbars in TV mode, no white flash on load, and the playlist position kept.

**Language**
- Default language is English; an admin can change it; a user can override it; every string, chart label, tooltip, export header and glossary entry exists in both languages or falls back to English; numbers and dates follow the language without changing the underlying values.

**TV display**
- All dashboards render correctly in the four viewports (half and full, at 100% and 200% scaling); on a TV, tooltip values appear as persistent labels; the CEO can add, remove, reorder and time dashboards for his display from his own device and the TV follows without reload; half or full presentation is applied per item; dashboards the viewer may not see are skipped.

**Database safety**
- No migration or code path writes outside `app_config`; business-table sessions are read-only; the app starts and works with a single database user; an optional read-only user can be added by configuration only; a restore of the `app_config` backup is tested.

**Departments**
- A user's department switch and role control navigation, API, AI tools, export and shared links; direct URL or API access to a disabled department is refused.
- Planned departments show "Coming soon" only to users who have them enabled; disabling a department hides it.
- A new department can be added through data (department entry and manifest) without changing shell code.

**Authentication**
- Only admin-created, enabled users can sign in; nothing is created automatically.
- Users sign in with username and password, must change a temporary password at first login, and can change it later.
- TOTP: when MFA is enabled for a user, enrollment and a valid code are required at every sign-in; a reused code is rejected; clock drift of one step is accepted; recovery codes work once.
- The app refuses to start without a valid master key in `.env`; keys and MFA secrets are unreadable in the database; key rotation re-encrypts all values.
- Repeated failures lock the account temporarily; resets and disables work; no password appears in logs.

**Auto-refresh**
- After an ETL run finishes, open dashboards and kiosk displays update without reload; the as-of time changes; caches never serve the previous version.
- A failed or still-running ETL shows the previous completed as-of and does not change numbers mid-run.

**MFA**
- Admin can enable or disable MFA per user; an enabled user must enroll and then enter a code at every sign-in; recovery codes work once; admin reset works; users cannot disable their own MFA; all changes are audited.

**AI provider settings**
- Admin can add, edit, test, activate and delete provider profiles; the API key is never returned by any API or written to a log; switching provider applies to the next question; non-local providers require the data-sharing acknowledgement.

**AI Chat**
- Every number equals the dashboard service output or an exact `calculate` result; unavailable or out-of-scope data gets a plain "not available"; typos and ambiguous names trigger a confirmation; no causes, forecasts or tables outside the approved ones.
- "Month to date" requests get the monthly-data explanation and the refresh time; chat data older than 60 days is deleted.
- Pop-up opens for visual answers, hides on close/background/other menu, stays in the chat, reopens on click.

**Data Findings**
- Mismatch between DIST count and `distaktif` total appears as a finding; numbers on dashboards are unchanged; findings respect user scope.

**Access control**
- A user limited to INA cannot read ASW data by any request; a user limited to one region sees only that region everywhere, including totals and cards.
- A user with full access sees both principals and switches with the buttons.
- Roles, capabilities and scopes can be changed in the admin screen without a deploy, and the change takes effect within the documented window.
- No role, principal, region or table name is hard-coded in the web or API code.

**All dashboards**
- Every bar, card, growth point and grid cell has a keyboard-reachable tooltip showing target, real and % (growth also last-year and growth %) (`DASH-001`).
- Month pickers drive all period filters (`DASH-003`).

**First dashboard**
- Date filters update all related widgets and reject invalid ranges.
- Role totals and IN/OUT percentages match owner-approved examples.
- Sell IN/Out match approved formulas including missing and zero targets.
- Working-day progress matches the approved calendar and is independent of sales achievement.
- Every row of the color test table passes; boundaries 69.99, 70, 99.99, 100 and above 100 are covered.
- Region navigation works by mouse and keyboard.
- Loading, empty, error and freshness states are understandable.

**Growth and upload dashboards**
- Window always equals the selected month and the 6 months before it, plus last year's same months; selecting the next month shifts it by one (`GROW-001`).
- Growth equals the approved formula; closed months show growth, the current month does not; zero last-year shows n/a.
- Upload cells match source rows (`Updated = 1`), and the row count equals the number of green cells.

**Editor and themes**
- View mode prevents accidental layout changes.
- Authorized editor can add/remove/move/resize widgets, edit text, set background, preview, save, cancel, reload.
- Invalid layout/settings are rejected safely.
- Light/dark/system themes readable and persist as intended.
- Chrome and Firefox layouts fit after sidebar collapse and at supported viewport sizes.

**Rotation**
- Admin picks dashboards, order, duration; default 10 s.
- Pause/resume/next/previous/exit work with no duplicate timers.
- Rotation blocked during editing or unsaved changes.
- Failed dashboard behavior is visible and tested.

**Engineering quality**
- CI passes lint, typecheck, unit tests, build, required E2E.
- No secrets, guessed production schema or hardcoded production metrics committed.
- Setup, environment variables, metric definitions, API contract and deployment notes documented.

## 3. Error, fallback and recovery scenarios

| Scenario | Required behavior |
|---|---|
| PostgreSQL/API unavailable | Actionable error and retry; never show zeros as valid |
| Empty result | Explain that no data matches the filters |
| Partial data / missing targets | "Unavailable" plus data-quality warning per approved policy |
| Invalid or obsolete layout version | Safe default layout, recoverable configuration error reported |
| Missing background asset | Theme surface fallback |
| Invalid threshold rules | Reject save; keep last valid published rules |
| Dashboard fails during rotation | Skip or keep last good (setting); non-blocking indicator |
| Session/permission expiry | Preserve safe draft if feasible; reauthenticate; reject unauthorized writes. Kiosk displays use a dedicated credential, not interactive login |
| Two editors conflict | Prompt to reload/compare; never silently lose changes |
| Resize or sidebar toggle | Charts reflow without page reload |

## 4. Testing strategy

1. **Pure unit:** formulas, threshold evaluation, date normalization, layout validation, playlist transitions.
2. **Repository/query:** approved SQL against a test database or controlled fixtures; validate grain and filters; double-counting test per metric.
3. **Component:** filters, KPI states, chart fallback, editor controls, settings forms.
4. **API and authorization:** schema validation, error shape, response contracts; for every endpoint, tests with users of different roles and scopes (out-of-scope principal or region, empty scope, full access, scope change, cache isolation between scopes).
5. **Browser E2E (Playwright, Chromium + Firefox):** date filters, region switching, theme, sidebar resize, edit/save/reload, playlist timing and controls.
6. **Accessibility:** automated axe plus manual keyboard and screen-reader review.
7. **Visual regression (optional):** after the visual design is agreed, using stable fixtures.
8. **Tableau parity:** run both systems on agreed sample periods and record differences; tolerance set by the owner (`[O]`).

Mock fixtures never assert real metrics as production truth. Each owner-approved example records inputs, intermediate values, final output and source/query provenance.

**Traceability:** every requirement ID links to implementing code and tests where practical. Prefixes: `DATA-`, `METRIC-`, `RULE-`, `UI-`, `EDIT-`, `PLAY-`, `SEC-`, `A11Y-`, `OPS-`.

## 5. Definition of ready (per metric)

A metric may go to production implementation when:
- Source table/view and columns are supplied (mapping row `Approved`).
- Grain and join keys are documented.
- Formula, denominator, date rules, rounding and null/zero behavior are approved.
- At least one independently verified example exists.
- Authorization and filter scope are understood.
- API DTO and UI display/empty/error behavior are defined.
- Tests can assert expected results without production customer data.

The shell, themes, navigation, editor scaffolding and mock-backed UI may proceed earlier if mock data is clearly labeled and no irreversible data-model assumptions are made.

### Later phases (not scheduled): further departments
**Gate (owner decision, DEC-074):** these start only after Release 1 (Sales) is finished, validated by the owner and working in use. The slots exist in the UI from Phase 1, but no module work begins before the gate.

- Marketing (product dashboards with product name and `pcode`; SE-level monthly target and sell-out; daily progress and near-real-time achievement per `PROG-001`), Finance, HR and others. Each is added as a department entry plus a module manifest, with its own approved data mapping, KPIs, scope dimensions and, for Finance and HR, the sensitivity rules in `10-departments-and-modules.md` section 6.
