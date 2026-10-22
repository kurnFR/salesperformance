# Roadmap, Acceptance and Testing

Related: [`PRD.md`](../PRD.md) · [`decisions.md`](decisions.md)

## 1. Roadmap

Each phase lists tasks and an exit gate. Work in small reviewable PRs, citing requirement IDs.

### Phase 0: Discovery and data contract
- Verify the Tableau-derived columns in PostgreSQL (`03-data-model.md` section 4.6) and approve the mapping rows. The workbook, screenshots and table names are already in the repo.
- Close blocking decisions in `decisions.md`: at least DEC-001 to DEC-004, DEC-006, DEC-020, DEC-021, DEC-027, DEC-028, DEC-030.
- **Exit:** approved metric and API contracts; no guessed schema.

### Phase 1: Foundation
- Monorepo, TypeScript strict, lint, test, CI; `AppShell`, sidebar, routing, theme provider, error boundaries, `/api/health`.
- Repository interfaces with mock implementations; `fixtures/` and golden cases.
- Authentication: admin-created local users (Argon2id, forced change of the temporary password, lockout, admin reset, change password) behind an `AuthProvider` interface. Microsoft sign-in is deferred.
- Access-control foundation: `AccessContext`, capability catalog, scope-required repositories, `GET /api/me/access`, seed roles as data (`02-architecture.md` section 10). Use a development identity stub until DEC-007 is decided.
- **Exit:** shell works in Chrome and Firefox; CI green; scope tests pass.

### Phase 2: First read-only Sales Performance dashboard
- Filters, team totals, role cards, region navigation, Sell IN/Sell Out chart, working-day indicator, display date, freshness.
- Connect PostgreSQL only after the relevant mapping rows are `Approved`.
- **Exit:** outputs match owner-validated examples and Tableau parity samples.

### Phase 3: Configurable achievement rules and access admin
- Admin screens for roles, capabilities, scopes and user assignments (`UI-019`), with audit.
- Admin settings, validation, preview, persistence, versioning, boundary tests from the color table.

### Phase 4: Dashboard editor
- Widget registry, drag/resize, add/remove/configure, text blocks, backgrounds, preview/save/revert, versioning, conflict handling.

### Phase 5: Playlist and presentation mode
- Playlist admin, ordering, durations, rotation controller, kiosk mode, playback controls, kiosk authentication.

### Phase 6: Growth and upload dashboards, then further dashboards
- Order: Growth Sell In, Growth Sell Out (shared matrix component), Daily Upload. Each is built separately and needs its mapping rows approved and acceptance examples (`06-dashboard-catalog.md`).
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

**Authentication**
- Only admin-created, enabled users can sign in; nothing is created automatically.
- Users sign in with username and password, must change a temporary password at first login, and can change it later.
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
