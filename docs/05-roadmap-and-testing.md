# Roadmap, Acceptance and Testing

Related: [`PRD.md`](../PRD.md) · [`decisions.md`](decisions.md)

## 1. Roadmap

Each phase lists tasks and an exit gate. Work in small reviewable PRs, citing requirement IDs.

### Phase 0: Discovery and data contract
- Fill the source-mapping table (`03-data-model.md` section 4); obtain the Tableau workbook and the screenshot into `docs/reference/`.
- Close blocking decisions in `decisions.md` (at least DEC-001 to DEC-004, DEC-006, DEC-007).
- **Exit:** approved metric and API contracts; no guessed schema.

### Phase 1: Foundation
- Monorepo, TypeScript strict, lint, test, CI; `AppShell`, sidebar, routing, theme provider, error boundaries, `/api/health`.
- Repository interfaces with mock implementations; `fixtures/` and golden cases.
- **Exit:** shell works in Chrome and Firefox; CI green.

### Phase 2: First read-only Sales Performance dashboard
- Filters, team totals, role cards, region navigation, Sell IN/Sell Out chart, working-day indicator, display date, freshness.
- Connect PostgreSQL only after the relevant mapping rows are `Approved`.
- **Exit:** outputs match owner-validated examples and Tableau parity samples.

### Phase 3: Configurable achievement rules
- Admin settings, validation, preview, persistence, versioning, boundary tests from the color table.

### Phase 4: Dashboard editor
- Widget registry, drag/resize, add/remove/configure, text blocks, backgrounds, preview/save/revert, versioning, conflict handling.

### Phase 5: Playlist and presentation mode
- Playlist admin, ordering, durations, rotation controller, kiosk mode, playback controls, kiosk authentication.

### Phase 6: Additional sidebar dashboards
- One at a time, each needing its own schema mapping, KPI definitions and acceptance examples.

### Phase 7: Production hardening
- Security review, accessibility audit, performance/load tests, observability, backup/restore, deployment runbook, user acceptance testing, Tableau cut-over plan.

## 2. Acceptance criteria

**First dashboard**
- Date filters update all related widgets and reject invalid ranges.
- Role totals and IN/OUT percentages match owner-approved examples.
- Sell IN/Out match approved formulas including missing and zero targets.
- Working-day progress matches the approved calendar and is independent of sales achievement.
- Every row of the color test table passes; boundaries 69.99, 70, 99.99, 100 and above 100 are covered.
- Region navigation works by mouse and keyboard.
- Loading, empty, error and freshness states are understandable.

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
4. **API:** schema validation, permissions, error shape, response contracts.
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
