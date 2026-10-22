# Instructions for AI Coding Agents

Read in this order: `PRD.md` -> `docs/decisions.md` -> the doc(s) relevant to your task. Treat the specification as requirements, not proof that code, frameworks or schema exist. The repository may be empty apart from docs.

## Hard rules

- Do not create or alter business tables, or guess table/column names, joins or formulas. Use mock repositories and `fixtures/` until the relevant source-mapping row in `docs/03-data-model.md` is `Approved`.
- Do not apply `app_config` migrations until DEC-009 is confirmed.
- Never commit secrets or real customer/sales data. Never hardcode production metrics, regions, headcounts, playlists or thresholds in presentation components.
- Keep metric, working-day and threshold logic in tested domain/API code (`packages/dashboard-core`, API services), never duplicated in UI.
- Do not merge or deploy without explicit authorization.
- Mock data must be labelled `MOCK` and is never evidence of business truth.

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
