# Sales Performance Dashboard — Product Requirements Document

**Status:** Draft v2 (living specification)  
**Repository:** https://github.com/kurnFR/salesperformance  
**Product:** Interactive web replacement for the existing Tableau Sales Performance dashboard

This file is the short entry point: what we are building, why, for whom, and how the rest of the specification is organised. Detail lives in the documents listed in [Document map](#document-map). The previous single-file PRD is preserved in git history (commit `15f4b3b`).

## 1. Summary

Build a maintainable, responsive BI web application backed by the existing PostgreSQL database. Reproduce the four supplied Tableau dashboards (Sales Performance, Growth Sell In, Growth Sell Out, Daily Upload), then add configurable layouts, filters, light/dark themes, dashboard playlists and administration incrementally. It must work in current Chrome and Firefox and adapt to viewport changes.

The owner has supplied the source table names (`table_name`), the Tableau workbook and screenshots (`docs/reference/`). Column names are derived from the workbook and **must be verified in PostgreSQL** before use. Do not guess anything beyond that, do not create business tables, do not fabricate production metrics. Use typed contracts and clearly marked mock fixtures until each item's mapping is approved.

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

## 3. Dashboards and features in scope

| # | Dashboard | Reference | What it shows | Spec |
|---|---|---|---|---|
| 1 | Sales Performance | `sheet1.png` | Team counts, role cards (RSM/ASM/SPV/DIST IN/OUT), Sell IN / Sell Out bars per region, working-day progress | `06` section 2 |
| 2 | Growth Sell In | `sheet2-growth sell in.png` | Rolling 7 months, this year vs last year, per region, growth line | `06` section 3 |
| 3 | Growth Sell Out | `sheet3-growth sell out.png` | Same layout using Sell Out | `06` section 3 |
| 4 | Daily Upload | `sheet 4-Daily Upload.png` | Per-distributor daily upload calendar for a month | `06` section 4 |
| 5 | Data Findings | (new) | Anomalies and findings to validate, such as DIST total vs manual active-distributor upload | `08` |
| 6 | Sales Detail (pivot) | (owner example, not a fixed layout) | The user chooses columns (hierarchy dimensions and fact values) over a period range, with totals, drill-down through the hierarchy down to SE and item where data exists, and export | `06` section 6 |
| 7 | Performance Drivers | (new, owner-requested insight dashboard) | Where the missing value is, Sell IN versus Sell OUT signal, persistence and momentum, status distribution and target ambition, year to date, Regular versus Festive product groups, network health, volume versus value and field coverage | `11` |
| - | Executive Summary | (new) | One-screen view for CEO, General Manager and Regional Manager: pace, regions, needs-attention, movers, data health | `09` |
| - | AI Chat | (new, right panel icon) | Answers questions from the data only, with pop-up charts that stay in the chat | `07` |

**Cross-cutting requirements (owner-confirmed unless marked):**
- Every chart element has a **tooltip** with target value, real value and % (`DASH-001`).
- Growth dashboards always show a **rolling 7 months** ending at the selected month (default: current month); next month the window moves with it (`DASH-002`).
- Period selection is **by month** (`DASH-003`, from Tableau).
- **Principals:** ASW and INA are chosen with buttons in the web; both are in release 1; access per user is set by admin (`DASH-004`, `DASH-008`, DEC-022).
- Shared visual style (dark navy, orange outlines) is an optional brand preset (`DASH-006`).
- **Language:** English and Bahasa Indonesia; default English; admin can change the default; users can switch (`I18N-001` to `I18N-007`).
- **CEO TV:** a mini PC on a 4K TV with two windows: the CCTV viewer and the dashboard browser. Default is the dashboard on half of the screen; the user can switch it to full by hand. The dashboard **auto-fits** the window (no blank space) in either size. The CEO chooses which dashboards rotate and how long, and remote-controls the TV (`PLAY-009` to `PLAY-013`, `UI-037`, `UI-041`).
- **Server:** Ubuntu 20.04 with Docker, Node.js 24, PostgreSQL 12.22; the app runs in Docker and is tested on PostgreSQL 12 and a current version (`OPS-022` to `OPS-024`). The ETL sometimes drops, truncates and adds columns, so dashboards serve last-good data during updates and tolerate schema changes (`OPS-017` to `OPS-021`).
- **Database:** the existing PostgreSQL, configured in `.env` (`.env.example` in the repository); app data in its own schema `app_config`; no second database (`OPS-012` to `OPS-016`).
- **Unmapped values:** some distributors or cities have no region. Dashboards exclude them (as Tableau does) but always show the amount; Sales Detail includes them as "(Unmapped)"; a data finding lists them (`DATA-006`, `METRIC-045` to `METRIC-049`).
- **Management use:** Executive Summary, regional view, drill-down, rankings, export and share, glossary, phone layout, saved views (`MGMT-001` to `MGMT-018`, `docs/09-management-experience.md`).
- **Planned with Marketing (after Release 1 is validated):** SE-level monthly target and daily sell-out values (they exist already), day-by-day achievement progress against the working-day pace (nightly refresh at first), and near-real-time refresh later (`PROG-001` to `PROG-005`, DEC-070, DEC-071).
- **Department-ready platform:** Sales is active now; Marketing (product dashboards), Finance, HR and others are planned. The UI and access model support departments from the start, with a department switcher, "Coming soon" pages and per-user department enable/disable and department roles (`DEPT-001` to `DEPT-018`, `docs/10-departments-and-modules.md`).
- **AI Chat** in the right panel answers only from database data, never assumes or invents, and shows results as pop-up charts that stay in the chat (`AI-001` to `AI-047`).
- **Data Findings** sheet lists anomalies to validate; manual-upload sources are labelled (`FIND-001`, `DASH-009`).
- **Sign-in:** admin creates users with a username and password; users can change their password after login. Microsoft sign-in is skipped for now (no Microsoft administrator) and can be added later (`AUTH-001` to `AUTH-011`).
- **Freshness and auto-refresh:** data is monthly, loaded by an ETL (about 3 hours, nightly today); the current month is month-to-date as of the last completed run, the refresh time is shown, and dashboards update automatically when a run finishes (`DASH-010`, `DASH-011`).
- **AI provider is a setting:** admins choose provider and model (9Router, OmniRoute, local model, Claude, Gemini, OpenAI/Codex, any OpenAI-compatible) and save the API key in settings (`AI-037` to `AI-047`).
- **MFA is configurable per user:** an admin enables or disables it for each user (`AUTH-010`).

## 4. Goals and non-goals

**Goals**
- Replace the Tableau dashboard using the existing PostgreSQL database as the source of truth.
- Match the reference: Start/End Date filters, role KPI cards, region navigation, Sell IN / Sell Out achievement bars, working-day progress, display date.
- Keep metric definitions consistent, server-calculated and testable.
- Let authorised editors add/remove widgets, drag and resize, edit text, and set backgrounds.
- Let admins configure achievement thresholds/colors without code changes.
- Let admins define roles, capabilities and data scopes (principal, region, other filters) per user, and switch features and filters on or off per role at any time, without code changes.
- Let users ask an AI assistant questions about the data, with answers grounded only in the data.
- Surface data anomalies for validation instead of hiding them.
- Provide an admin-managed playlist that rotates dashboards automatically (default 10 s each).
- Build each sidebar dashboard incrementally on shared components and a shared design system.

**Non-goals for the first release**
- Replacing PostgreSQL or changing its business schema.
- Hard-coded roles, principals or regions anywhere in code (all come from admin-managed configuration or data).
- Guessing joins, column names, KPI formulas or role-label meanings.
- A general-purpose arbitrary SQL editor.
- Building every future dashboard in the first milestone.
- Export (CSV/PNG/PDF), drill-through, annotations and alerting (candidates for later phases; confirm).

## 5. Users, roles and permissions (configurable)

**Nothing about access is hard-coded** (DEC-011, DEC-022). Admins define roles in an admin screen; each role has capabilities and a data scope; each user gets one or more roles.

- **Principal:** ASW and INA buttons in the web. A user sees only the principals their role allows. Only users granted full access can use both.
- **Filters such as region:** each user is allowed one region, several, or all, set by admin. The same mechanism covers other filters (branch, dashboard visibility).
- **Capabilities** (what a role may do) are checked by the server: view dashboards, edit layouts, manage achievement rules, manage playlists/assets, manage access. Which role holds which capability is data.
- **All numbers** (team totals, cards, bars, active distributors) are computed inside the user's scope.
- **Deny by default:** a user with no role or an empty scope sees nothing.
- **Seed roles** for first use are ordinary editable rows (suggested: Viewer, Display (kiosk), Editor, Administrator); a technical administrator handles secrets and deployment outside the product UI.

Design details: `docs/02-architecture.md` section 10; stored shapes: `docs/03-data-model.md` sections 2.5, 2.6 and 3. Sign-in is admin-created local username/password in release 1; Microsoft sign-in is deferred (DEC-007, `docs/02-architecture.md` section 11).

**Suggested seed capability sets** (`[P]`, editable by admins, shown only as a starting point):

| Capability | Viewer | Display | Editor | Administrator |
|---|:-:|:-:|:-:|:-:|
| `dashboard.view`, change permitted filters, presentation mode | Yes | Yes | Yes | Yes |
| `layout.edit` (widgets, text, backgrounds) | - | - | Yes | Yes |
| `rules.manage`, `playlist.manage`, `assets.manage` | - | - | - | Yes |
| `access.manage` (roles, scopes, users) | - | - | - | Yes |

## 6. Requirement authority

Every requirement carries a tag:

- `[C]` **Owner-confirmed** (explicit confirmation or approved data dictionary/example)
- `[R]` **Reference-observed** (visible in the Tableau screenshot; not a full business spec)
- `[P]` **Proposed default** (recommendation awaiting owner approval)
- `[O]` **Open decision** (needs owner data or policy)
- `[I]` **Implementation detail** (engineering choice; adjustable)

The screenshot is evidence of visual arrangement, not of schema, formulas or responsive behaviour. Mock data is never business evidence. Requirements have stable IDs with these prefixes: `DATA-`, `METRIC-`, `RULE-`, `UI-`, `EDIT-`, `PLAY-`, `SEC-`, `A11Y-`, `OPS-`, `DASH-` (cross-cutting dashboard rules), `PERF-` (dashboard 1), `GROW-` (dashboards 2 and 3), `UPLD-` (dashboard 4), `FIND-` (data findings), `AI-` (AI chat), `AUTH-` (authentication), `MGMT-` (management experience), `PIVOT-` (Sales Detail), `PROG-` (daily progress), `DEPT-` (departments and modules), `INS-` (insight dashboard), `OPS-` (operations).

## 7. Roadmap at a glance

| Phase | Outcome | Needs from owner |
|---|---|---|
| 0 | Discovery, approved data contract and decisions | Schema check report approved (DEC-076), ETL readiness semantics (DEC-077) |
| 1 | App shell, routing, themes, health check, mock fixtures, admin-created local sign-in with per-user TOTP, `.env` master key, access-control foundation, department-ready navigation | — |
| 2 | Sales Performance dashboard and Executive Summary on real data for INA and ASW: principal buttons, scoped data, tooltips, working-day progress from `dimdate`, auto-refresh from the ETL log, drill-down | Approved mapping rows and verified examples; ETL log table details (DEC-048) |
| 3 | Configurable achievement rules and the access admin screens (roles, scopes, feature/filter toggles, MFA per user) | Rule defaults confirmed |
| 4 | Dashboard editor | Persistence approval |
| 5 | Playlist and presentation mode | Kiosk auth decision |
| 6 | Sales Detail (pivot) with export, Growth Sell In, Growth Sell Out, Daily Upload, then Performance Drivers, then further dashboards | Per-dashboard mapping rows approved; insight decisions DEC-085 to DEC-090 |
| 7 | AI Chat and Data Findings | Findings thresholds (DEC-043); alert channels if wanted (DEC-055) |
| 8 | Production hardening | — |
| 9+ | Further departments (Marketing with product `pcode` and SE daily progress, Finance, HR, others), each added as data plus a module, **only after Release 1 is finished and validated** | Per department: approved schema, KPIs, scope dimensions, owners |

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
| `docs/11-insight-dashboard.md` | Performance Drivers: gap analysis of unused data, views, requirements, tests |
| `docs/10-departments-and-modules.md` | Department platform: model, module manifest, switcher and "Coming soon" UI, admin screens, sensitive departments |
| `docs/09-management-experience.md` | Management review: gaps found, personas, Executive Summary, derived metrics, export/share |
| `docs/07-ai-chat.md` | AI chat: grounded-answer design, tools, pop-up behavior, rules, tests |
| `docs/08-data-findings.md` | Data findings sheet: checks, record, workflow |
| `docs/06-dashboard-catalog.md` | The four dashboards: layout, widgets, tooltips, rolling window, per-dashboard requirements |
| `docs/decisions.md` | Decision log with proposed defaults and status |
| `docs/reference/` | Screenshots, Tableau workbook, extracted Tableau formulas |
| `table_name` | Data dictionary: CSV name to PostgreSQL table, per principal |
| `AGENTS.md` | Rules and workflow for AI coding agents |
