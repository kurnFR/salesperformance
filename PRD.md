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
