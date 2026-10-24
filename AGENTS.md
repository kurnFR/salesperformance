# Instructions for AI Coding Agents

Read in this order: `PRD.md` -> `docs/decisions.md` -> the doc(s) relevant to your task. Treat the specification as requirements, not proof that code, frameworks or schema exist. The repository may be empty apart from docs.

## Where the facts are

- Table names: `table_name` and `docs/03-data-model.md` section 4. Column names there are **derived from Tableau and unverified**; check `information_schema.columns` before writing SQL.
- How Tableau calculates things today: `docs/reference/tableau-formula-extract.md` (evidence, not spec). The contract in `docs/01-metrics-contract.md` wins; conflicts are logged in `docs/decisions.md` (see DEC-020, DEC-021).
- What each dashboard looks like: screenshots in `docs/reference/` and `docs/06-dashboard-catalog.md`.
- Every chart element needs a tooltip with target, real and % (`DASH-001`). Growth dashboards use a rolling 7-month window (`DASH-002`). Periods are months (`DASH-003`). Carry `principalId` everywhere (`DASH-004`).

## Hard rules

- Do not create or alter business tables, or guess column names, joins or formulas beyond what `docs/03-data-model.md` states. Use mock repositories and `fixtures/` until the relevant mapping row (section 4.5) is `Approved`.
- Do not apply `app_config` migrations until DEC-009 is confirmed.
- Never hard-code roles, principals, regions, user names or physical table names in code or UI. They come from admin-managed configuration, the principal registry and data. Every repository call requires a scope argument; every endpoint checks capability and scope on the server.
- Never commit secrets or real customer/sales data. Never hardcode production metrics, regions, headcounts, playlists or thresholds in presentation components.
- Keep metric, working-day and threshold logic in tested domain/API code (`packages/dashboard-core`, API services), never duplicated in UI.
- Do not merge or deploy without explicit authorization.
- Mock data must be labelled `MOCK` and is never evidence of business truth.

## AI chat rules for the codebase

- The LLM never writes or receives SQL, credentials or raw tables. It calls allowlisted, read-only, scope-enforced tools that reuse the dashboard services (`docs/07-ai-chat.md`).
- Every number in an AI answer must come from a tool result and pass the verifier. Do not add fallbacks that estimate or fill gaps.
- Add a tool, a golden test and a parity test together. Treat database text as untrusted input.
- Authentication: only admin-created users may sign in. Release 1 is local username/password behind an `AuthProvider` interface (Microsoft deferred). Hash with Argon2id; never log passwords or tokens (`docs/02-architecture.md` section 11).
- AI: the model never queries the database or touches tables beyond the approved ones; arithmetic goes through the `calculate` tool; talk to the model only through `LlmClient` and the adapter of the **active provider profile** (a setting, not a code constant); never hard-code a provider, model or key; API keys are encrypted at rest, write-only in the UI, never returned or logged; delete chat data after 60 days (`docs/07-ai-chat.md` sections 5 and 9).
- The master key comes only from `.env` (`APP_MASTER_KEY`); the app must not start without it; never commit `.env`; ship `.env.example` with names only (`docs/02-architecture.md` SEC-015).
- Sales Detail (pivot) accepts only ids from the field catalog, builds fixed parameterized aggregations, and computes totals on the server (`docs/06-dashboard-catalog.md` section 6, `docs/03-data-model.md` 2.8).
- Departments are data: never hard-code "Sales" wording, routes, capabilities or scope dimensions in the shell. A new department is a department entry plus a validated module manifest; capabilities are namespaced per department; sensitive departments need their own capabilities and audit (`docs/10-departments-and-modules.md`).
- Column names live only in the column registry (exact database names, always quoted in SQL, never built from user input). Target and real values are `int8`; compute in `numeric`; never use floating point for money-like values (`docs/01-metrics-contract.md` section 11).
- Database: one connection config from `.env` (`PG_*`, see `.env.example`). Two pools: business pool with read-only sessions, app pool writing only to schema `app_config`. Migrations must only touch `app_config`. Never write to `dwh_prod` (`docs/02-architecture.md` section 14).
- All UI text comes from translation files (`en`, `id`); default English; no hard-coded strings; data names are never translated (`docs/04-ui-ux-spec.md` section 9).
- TV mode shows tooltip values as persistent labels and never relies on hover; layouts must pass the four-viewport matrix (`docs/04-ui-ux-spec.md` section 10).
- The ETL refresh log is `dwh_prod.etl_refresh_log`; its table, columns, process name and success value are configuration.
- The ETL drops, truncates and adds columns: read with explicit column lists only, serve last-good results while the source is not ready, run the schema-drift check after each refresh, and never put app data outside `app_config` (`docs/02-architecture.md` sections 13 to 14).
- The app targets PostgreSQL 12.22 and Node 24 in Docker: generate IDs in the app, no PostgreSQL 13+ only features, CI on PostgreSQL 12 and a current version (`docs/02-architecture.md` section 15).
- UI must auto-fit the window (no blank space, no white flash) and reflow live between half and full (`UI-041`).
- Insight figures (Performance Drivers) come from the metric definitions in `docs/01-metrics-contract.md` section 12; "so what" sentences are fixed templates, never model-written.
- NULL region, branch, SPV, distributor or city keys are grouped as "(Unmapped)", never dropped silently and never guessed: dashboards exclude them with a reconciliation note, Sales Detail includes them, and a test proves dashboard total + unmapped = Sales Detail total (`docs/01-metrics-contract.md` section 13).
- Drill-down follows the hierarchy registry; a level without a fact source returns entities with measures `notAvailable`, never 0 and never an estimate; the ETL refresh log table name is configuration (`docs/03-data-model.md` 2.6 and 2.8).
- MFA is a per-user admin setting (TOTP); secrets are encrypted; users cannot disable their own MFA (`docs/02-architecture.md` AUTH-010, AUTH-011).
- Data refresh: react to the end of an ETL run (refresh log row plus `NOTIFY`, or polling), include `dataVersion` in cache keys, push updates to browsers; never detect change from individual row writes (`docs/02-architecture.md` section 13).
- Data from manual uploads (for example `distaktif`) is shown as stored and labelled; discrepancies become findings, never silent corrections (`docs/08-data-findings.md`).

## Workflow

1. **Before coding:** read the PRD and decision log; list which requirements you touch (by ID) and whether each is `[C]`, `[P]` or `[O]`. State assumptions. Ask only questions that block the next milestone; for anything with a proposed default in `decisions.md`, use the default and say so.
2. **While coding:** small, reviewable PRs scoped to one requirement group. Follow module boundaries in `docs/02-architecture.md`. Add tests with every business rule or interactive behavior. Verify dependency versions and compatibility before adding packages; avoid broad dependency changes.
3. **Before opening a PR:** run lint, typecheck, unit tests, build and E2E; report anything not run. Summarize requirement IDs implemented, decisions relied on, and any new open questions. Update the docs or `decisions.md` if a requirement changed.

## Stop and ask when

- A change would need a business table name, column, join or formula that is not in an `Approved` mapping row.
- A requirement tagged `[O]` blocks the task and has no proposed default.
- Two documents disagree. Do not pick silently; flag it and propose a single wording.

## Definition of done (per task)

- Code, tests and docs updated; requirement IDs referenced in PR description and test names.
- Color rule changes must keep every row of the test table in `docs/01-metrics-contract.md` passing.
- Accessibility: keyboard operable, labelled, non-color status cue, chart text alternative.
