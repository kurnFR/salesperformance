# Decision Log

Every open question has a **proposed default** so work can continue; the default applies only until the owner decides. Update status and date when a decision is made, and cite the DEC ID in PRs.

Status: `Confirmed` · `Proposed` (default in use, awaiting owner) · `Open` (blocked, no default) · `Superseded`.

## Confirmed

| ID | Decision | Confirmed | Where used |
|---|---|---|---|
| DEC-005 | Color rule: if achievement >= working-day progress -> green (even below 70%); otherwise <70% red, 70% to <100% yellow, >=100% green. Equality is green. Yellow overrides red when behind pace but >=70% | 2026-10-22 (owner, in chat) | `01-metrics-contract.md` section 4 |
| DEC-005a | Working-day progress is capped to 0-100% on the server | 2026-10-22 | `01-metrics-contract.md` 5.7 |
| DEC-005b | Working-day progress comes from the working-day table/calendar, not sales achievement | Earlier (owner) | `01-metrics-contract.md` 5.7 |
| DEC-005c | Role card example: RSM (5), IN 20% / 1 = one of five with achievement >= 100% | Earlier (owner) | `01-metrics-contract.md` 5.3 |

## Blocking Phase 0 and 2

| ID | Question | Proposed default | Impact if wrong | Blocks | Status |
|---|---|---|---|---|---|
| DEC-001 | PostgreSQL schema: sales facts, targets, regions, people/roles, working days | None; fill `03-data-model.md` section 4. Ask for Tableau `.twbx` | Entire data layer | Phase 2 | Open |
| DEC-002 | Are DIST and SE distinct? Which appear on cards vs header? | Treat as distinct until told otherwise | Wrong cards/totals | Phase 2 | Open |
| DEC-003 | Org hierarchy (RSM > ASM > SPV > SE) and how targets/achievement roll up | None | Wrong role achievement | Phase 2 | Open |
| DEC-004 | Achievement formulas, aggregation order, zero/missing target, denominator for people without data | Aggregate then divide; missing target = unavailable; missing-data people excluded from denominator but flagged | Wrong percentages | Phase 2 | Proposed |
| DEC-004a | Do IN/OUT mean Sell IN/Sell Out? Does exactly 100% count? | Yes, and yes (>=100%) | Wrong counts | Phase 2 | Proposed |
| DEC-006 | Business timezone and working-day semantics (as-of date, holidays, past/future periods, zero working days) | Business timezone confirmed by owner (placeholder `TBD`); past period 100%, future 0%, zero working days unavailable | Wrong progress, wrong colors | Phase 2 | Proposed |
| DEC-006a | Default date range on first load | Current calendar month to date | Confusing first view | Phase 2 | Proposed |

## Platform and operations

| ID | Question | Proposed default | Blocks | Status |
|---|---|---|---|---|
| DEC-007 | Identity provider for users | Organization SSO via OIDC if available; otherwise local accounts behind HTTPS; dev uses a stub role switcher | Phase 4 | Open |
| DEC-008 | Hosting and approved image storage | Single container deployment; curated server-side asset library in v1 (no arbitrary uploads) | Phase 4 | Open |
| DEC-009 | Where app-owned config lives | Separate `app_config` schema in the same PostgreSQL, created by migrations (`03-data-model.md` section 3) | Phase 3 | Proposed |
| DEC-010 | Do filters persist across rotation or per dashboard? | Period filter shared; other filters per dashboard | Phase 5 | Proposed |
| DEC-011 | Row-level data scope per user (for example an RSM sees only their region) | No row-level scope in v1; all viewers see all regions | Phase 2 | Open |
| DEC-012 | Kiosk display authentication | Dedicated long-lived read-only display token, revocable by admin | Phase 5 | Proposed |
| DEC-013 | UI language, number/date formatting, single vs multiple timezones across regions | Single business timezone; language confirmed by owner | Phase 1 | Open |
| DEC-014 | Data refresh cadence, staleness threshold, caching, read replica or materialized views | Query on demand with short server cache (60 s); stale after 2x source cadence | Phase 2 | Proposed |
| DEC-015 | Layouts/playlists scope: global, per role, per user, per display | Global layouts; one default playlist; per-display playlists later | Phase 4 | Proposed |
| DEC-016 | Rotation minimum duration and failure behavior | Minimum 5 s; on failure skip with notice | Phase 5 | Proposed |
| DEC-017 | Target displays (resolution, aspect, viewing distance) and minimum viewport | 1920x1080 and 1366x768 desktop; tablet-width reflow | Phase 1 | Open |
| DEC-018 | Tableau parity: sample periods and tolerance | 3 recent months; exact match after rounding | Phase 2 exit | Open |
| DEC-019 | Out-of-scope features: export, drill-through, annotations, alerts | All deferred past v1 | Phase 6 | Proposed |
