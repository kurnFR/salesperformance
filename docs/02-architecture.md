# Architecture

Related: [`PRD.md`](../PRD.md) · [`01-metrics-contract.md`](01-metrics-contract.md) · [`03-data-model.md`](03-data-model.md)

The repository currently contains no application code, so this is a greenfield project. The stack below is the **chosen starting point** (`[I]`); change it only with a recorded decision. Verify current package versions and compatibility at Phase 1 start and pin them in the lockfile; do not copy version numbers into documents.

## 1. System context

```mermaid
flowchart LR
  U[Viewer / Editor / Admin] --> W[Web app<br/>React + Vite]
  K[Kiosk display] --> W
  W -->|HTTPS JSON| API[API<br/>Fastify]
  subgraph Server
    API --> AZ[Auth + permissions]
    API --> SV[Services / domain rules]
    SV --> RP[Repositories]
  end
  RP -->|read-only, parameterized| SRC[(PostgreSQL<br/>business source of truth)]
  RP -->|read/write| CFG[(app_config schema<br/>layouts, rules, playlists)]
  SV --> AST[(Approved asset storage)]
```

Rules: no SQL in the browser; no database credentials in the browser; business tables are read-only for the app; app-owned configuration lives in a separate schema (see `03-data-model.md`).

## 2. Stack

**Frontend:** Node.js current LTS (pinned) · Vite + React + TypeScript (strict) · Tailwind CSS · shadcn/ui on Radix primitives · lucide-react · React Router · TanStack Query (server state, caching, cancellation) · Zustand (client-only state: editor selection, theme UI, rotation) · React Hook Form + Zod · Apache ECharts + echarts-for-react · react-grid-layout behind a local adapter · date-fns (presentation only) · one typed fetch wrapper (ky or native, pick one).

**Backend:** Fastify + TypeScript · `pg` with pooling, timeouts, bounded results · Drizzle ORM + drizzle-kit for app-owned tables only (`app_config`) · Zod at request, response and config boundaries · Pino with redaction · `@fastify/helmet`, `@fastify/cors`, `@fastify/rate-limit` · `@fastify/swagger` + UI for API contracts · least-privilege DB credentials.

**Export:** Excel and CSV generated on the server with a maintained spreadsheet library (verify version at Phase 1); PDF and image snapshots rendered from a print-optimised page with headless Chromium (Playwright is already in the stack). Exports run inside the user's scope and add filters, scope note and data as-of to the file.

**Quality and delivery:** pnpm workspaces · Vitest, Testing Library, user-event, jest-axe · Playwright (Chromium and Firefox) · ESLint, typescript-eslint, Prettier · CI: frozen-lockfile install, lint, typecheck, unit tests, build, E2E.

**Do not add** Redux, a second charting library, overlapping form libraries or a second ORM without a documented need.

## 3. Repository layout (proposed)

```
apps/web/                  React + Vite UI
apps/api/                  Fastify API (routes, services, repositories)
packages/contracts/        Shared DTOs and Zod schemas
packages/dashboard-core/   Metric types, layout schema, threshold evaluation, playlist state machine
packages/config/           Shared tooling config
deploy/                    Dockerfile, docker-compose.yml, backup and restore scripts, TV setup guide
docs/                      Specification (this folder)
fixtures/                  Clearly labelled MOCK data and golden test cases
```

Modular monolith, not microservices. Route handlers stay thin. Never duplicate working-day or metric formulas across layers.

## 4. Layering

| Layer | Responsibility |
|---|---|
| Route | Authentication/authorization, input parsing, response mapping |
| Service / domain | Business rules, calculations, state-transition validation |
| Repository | Parameterized PostgreSQL access, query-specific aggregation |
| Contract | Request/response DTOs, Zod schemas, stable error shape |
| Configuration | Environment variables, safe runtime defaults |

**Repository interface pattern:** services depend on interfaces (for example `MetricsRepository`). Provide a `MockMetricsRepository` (reads `fixtures/`) now and a `PgMetricsRepository` after the schema is approved. This lets the UI and API be built and tested end to end without guessing the schema.

## 5. Frontend component boundaries

| Component | Responsibility |
|---|---|
| `AppShell` | Layout, sidebar, top bar, route outlet |
| `DashboardRoute` | Loads dashboard metadata, route-level states |
| `GlobalFilterBar` | Date filters and shared filter state |
| `DashboardGrid` | Renders layout from validated widget definitions |
| `WidgetRenderer` | Resolves an allowlisted widget type to a component |
| `KpiCard`, `AchievementBarChart`, `WorkingDayProgress` | Presentational only; receive calculated values and statuses |
| `DashboardEditor` | Drag/resize/add/remove, layout draft state |
| `ThemeProvider` | Light/dark/system tokens and persistence |
| `RotationController` | Playlist state machine and timing, independent of charts |
| `AdminSettings` | Validated forms for rules and playlists |

Domain modules own: metric DTOs and formatting, achievement formulas, role count logic, working-day semantics, threshold evaluation, layout schema/versioning, playlist validation and transitions. Presentation components never query PostgreSQL, contain SQL, infer formulas, or own rotation timers.

## 6. API contract (proposed; none exist yet)

| Method and path | Purpose | Permission |
|---|---|---|
| `GET /api/health` | Liveness/readiness | Public |
| `GET /api/departments`, `GET/POST/PUT /api/admin/departments`, `PUT /api/admin/users/:id/departments` | Department list for the switcher; admin department management; per-user department enable/disable and roles (`10-departments-and-modules.md`) | Authenticated / `access.manage` |
| `GET /api/me/access` | Signed-in user's capabilities and data scope (allowed principals, regions, dashboards); drives which buttons and filter options the UI shows | Authenticated |
| `GET/POST/PUT/DELETE /api/admin/ai-providers`, `POST /api/admin/ai-providers/:id/test`, `PUT /api/admin/ai-settings` | AI provider profiles, API key (write-only), tests, global AI settings | `ai.settings.manage` |
| `GET /api/data-version`, `GET /api/events` (server-sent events) | Current data version and as-of; push notice when a data refresh finishes | Authenticated |
| `GET/POST/PUT /api/admin/roles`, `/api/admin/users/:id/roles`, `/api/admin/scopes` | Manage roles, capabilities, data scopes and user assignments | `access.manage` |
| `GET /api/dashboards` | List dashboards | Viewer+ |
| `GET /api/dashboards/:dashboardId` | Dashboard definition + active layout | Viewer+ |
| `GET /api/sales-performance/summary?principalId=&startMonth=&endMonth=&regionId=` | Team totals, role cards | Viewer+ |
| `GET /api/sales-performance/regions?principalId=&startMonth=&endMonth=&regionId=` | Sell IN / Sell Out per region with statuses | Viewer+ |
| `GET /api/calendar/working-day-progress?month=&asOfDate=` | Working-day progress + metadata | Viewer+ |
| `GET /api/sales-performance/growth?principalId=&month=&window=7&metric=sellIn\|sellOut` | Rolling-window growth matrix (`DASH-002`) | Viewer+ |
| `GET /api/sales-performance/active-distributors?principalId=&month=` | Active / total distributors per region | Viewer+ |
| `GET /api/executive-summary?principalId=&month=` | Executive Summary blocks (headline, tiles, needs-attention, movers, data health) within scope (`09-management-experience.md`) | Viewer+ |
| `POST /api/sales-detail/query` (read-only; body: period range, dimensions, measures, filters, mode flat or drill, drill path, sort, page) | Pivot-style detail with totals and on-demand children; validated against the field catalog (`PIVOT-011`, `PIVOT-014`) | Viewer+ |
| `GET/POST/PUT /api/displays`, `GET/PUT /api/displays/:id/playlist`, `POST /api/displays/:id/control` (next, previous, pause, resume, jump) | Display profiles (for example the CEO TV), their playlists, and remote control from a phone or laptop (`PLAY-009` to `PLAY-013`) | `display.manage` for that display |
| `GET /api/sales-detail/hierarchy` | Hierarchy levels, which measures each level supports, deepest level the user may open | Viewer+ |
| `POST /api/export/sales-detail`, `POST /api/export/snapshot` | Excel/CSV of a table; PDF/image snapshot of a dashboard (`MGMT-007`) | `export.use` |
| `GET/PUT /api/me/views`, `GET /api/glossary` | Saved views; metric glossary generated from the metrics contract | Authenticated |
| `GET /api/uploads/daily?principalId=&month=&regionId=&branchId=&kind=sales\|stock` | Daily upload grid | Viewer+ |
| `GET/PUT /api/settings/achievement-rules` | Rule set | Admin |
| `GET/POST/PUT /api/playlists` | Playlists | Admin (read: Viewer+) |
| `PUT /api/dashboards/:id/layout` with `If-Match: <version>` | Save layout (optimistic concurrency) | Editor+ |
| Asset list/upload | After storage is approved | Admin |
| `POST /api/ai/chat`, `GET /api/ai/sessions`, `GET /api/ai/results/:id`, `POST /api/ai/results/:id/refresh` | AI data chat (`07-ai-chat.md`) | `ai.chat.use` |
| `GET /api/findings`, `PUT /api/findings/:id` | Data findings (`08-data-findings.md`) | `findings.view` / `findings.manage` |
| `POST /api/auth/login`, `POST /api/auth/mfa/verify`, `POST /api/auth/mfa/enroll`, `PUT /api/admin/users/:id/mfa` (enable, disable, reset), `POST /api/auth/change-password`, `POST /api/auth/logout` | Local login, password change, sign-out (Microsoft callback added only when that provider is) | Public / Authenticated |

Every endpoint documents purpose, permissions, filters, response fields, errors, cache policy and freshness. Never return SQL, connection strings, stack traces or internal sensitive identifiers.

**Response envelope for metric endpoints:**

```json
{
  "data": { },
  "period": { "startMonth": "2026-10", "endMonth": "2026-10", "timezone": "TBD" },
  "filtersApplied": { "principalId": "INA", "regionId": null },
  "dataAsOf": "2026-10-21T23:00:00Z",
  "generatedAt": "2026-10-22T05:31:00Z",
  "warnings": [ { "code": "MISSING_TARGET", "scope": "region-002" } ]
}
```

**Error shape (stable):**

```json
{ "error": { "code": "VALIDATION_ERROR", "message": "Start date must not be after end date.", "requestId": "..." } }
```

Validate configuration on read and write; malformed persisted configuration falls back safely and logs an operational warning.

## 7. Rotation state machine

One controller, one timer. Clean up timers/listeners on every transition.

```mermaid
stateDiagram-v2
  [*] --> Idle
  Idle --> Playing: start
  Playing --> Paused: pause
  Paused --> Playing: resume
  Playing --> Playing: timer elapsed / next / previous
  Playing --> Loading: advance to next item
  Loading --> Playing: loaded
  Loading --> Playing: load failed, skip or keep last good, show notice
  Playing --> Idle: exit
  Paused --> Idle: exit
  Idle --> Blocked: edit mode or unsaved changes
  Blocked --> Idle: edit finished
```

## 8. Layout save flow (concurrency)

```mermaid
sequenceDiagram
  participant E as Editor UI
  participant A as API
  participant D as app_config
  E->>A: PUT layout (If-Match: version N)
  A->>A: authorize editor + validate layout schema
  A->>D: insert version N+1 if current == N
  alt current version is N
    D-->>A: ok
    A-->>E: 200 version N+1
  else someone saved first
    D-->>A: conflict
    A-->>E: 409 -> prompt to reload/compare, never overwrite
  end
```

## 9. Security, performance, observability

- `SEC-001` `[C]` Permissions enforced on the server; filters and identifiers allowlisted; all queries parameterized.
- `SEC-002` `[C]` Credentials stay server-side; metric queries use a read-only least-privilege account.
- `SEC-003` `[P]` Restrictive CORS, security headers, rate limits, request timeouts, safe errors, redacted logs.
- `SEC-004` `[P]` Validate image uploads (type, size, dimensions); no unsafe external URLs, path traversal or arbitrary HTML.
- `SEC-005` `[O]` Authentication provider and kiosk credential model (DEC-007, DEC-012).
- `OPS-001` `[P]` Aim for LCP <= 2.5 s; set API latency targets once data volume and hosting are known.
- `OPS-002` `[P]` Fetch active dashboard data first; prefetch only the next rotation item when safe.
- `OPS-003` `[P]` Structured logs with request IDs, health endpoint, error tracking, query timing.

## 10. Access control: configurable, not hard-coded (DEC-011, DEC-022, DEC-036)

**Owner requirement:** the web has principal buttons (ASW, INA). Admins decide, per role and per user, which principals, regions and other filters each person can use. Some users see one principal, some both (full access); some see one region, some several. Roles and permissions are managed in an admin screen and stored as data.

### 10.1 Model

- **Departments:** capabilities are namespaced per department (for example `sales.dashboard.view`), a role belongs to a department, and each user has each department switched on or off (`10-departments-and-modules.md`). A department enabled with no role gives no access.
- **Capabilities** are the things the code checks: actions (`layout.edit`, `rules.manage`, `playlist.manage`, `access.manage`, `findings.manage`, `ai.settings.manage`, `mfa.manage`, `export.use`), **features** (`dashboard.view` per dashboard, `ai.chat.use`, `findings.view`) and **filters** (`filter.principal`, `filter.region`, `filter.branch`, and so on). The catalog is fixed in code because code must check each item (a brand-new feature registers its capability when it ships); **who has each one is data**, and admins can turn any of them on or off per role at any time.
- **Filter policy per role:** each filter is `hidden`, `locked` (fixed to the role's scope value) or `editable` within the allowed values.
- **Roles** are created, renamed and deleted by admins. A role = a set of capabilities + a data scope.
- **Data scope** is, per scope dimension, either `all` or an explicit list of values. Dimensions come from a registry: `principal`, `region`, `branch`, `dashboard` (extendable without changing route code).
- **"Full access"** is `principal: all` (and `region: all`) on a role, not a hard-coded role name.
- **Users** get one or more roles; an optional per-user override can add or remove scope values.
- **Effective access** = union of the user's roles (capabilities and scope values per dimension), then user overrides. No role, or an empty scope, means **no access** (deny by default).
- Principals, regions and branches are values read from approved data or the principal registry (`03-data-model.md` 2.6). They are never listed in code or in UI components.

### 10.2 Enforcement (server only)

```mermaid
flowchart TD
  R[Request] --> AU[Authenticate user]
  AU --> AC[Build AccessContext<br/>capabilities + scope per dimension]
  AC --> CP{"capability<br/>allowed?"}
  CP -- no --> D403[403 Forbidden]
  CP -- yes --> VS{"requested principal/region<br/>inside scope?"}
  VS -- no --> D403
  VS -- yes --> SV[Service: apply scope to every query]
  SV --> RP[Repository requires a scope argument]
  RP --> DB[(PostgreSQL)]
  RP --> OUT[Response computed only from in-scope rows]
```

- `SEC-006` `[C]` Permissions and data scope are resolved on the server for every request; hiding buttons in the UI is never the control.
- `SEC-007` `[C]` Repositories take a mandatory scope argument; there is no code path that queries business tables without it.
- `SEC-008` `[C]` All totals, role cards, bars and active-distributor figures are computed within scope (`01-metrics-contract.md` section 8). When a filter is not supplied, the default is "all values the user is allowed", never "everything".
- `SEC-009` `[P]` Cache keys include a hash of the scope, so one user's cached result is never served to a user with a different scope.
- `SEC-010` `[P]` Changes to roles, capabilities, scopes and user assignments are audited (who, what, when) and take effect on the next request or within a short, documented cache window.
- `SEC-011` `[C]` The admin UI manages roles, capabilities, scopes and assignments with validation (cannot remove the last user holding `access.manage`).
- `SEC-012` `[P]` Seed roles for first use (for example Viewer, Editor, Administrator, Display) are ordinary editable data, not special-cased in code.
- `SEC-013` `[C]` Users are created by an admin and sign in with local credentials in release 1 (section 11); permissions key off the app's own user id, so adding Microsoft later does not change roles or scopes.
- `SEC-015` `[C]` **Master key in `.env`.** The administrator sets the encryption master key in the server's `.env` file (for example `APP_MASTER_KEY`, a random 32-byte value). The app refuses to start if it is missing or invalid. It is never stored in the database, never logged, never sent to a client. `.env` is git-ignored (verified in `.gitignore`) and the repository ships `.env.example` with names only. File permission restricts it to the service account. It encrypts AI API keys and MFA secrets with AES-256-GCM; each encrypted value carries a key id so the key can be rotated (re-encrypt command, then swap). **If the key is lost, saved API keys must be entered again and users must re-enroll MFA**, so the administrator keeps a secure backup separate from database backups (DEC-049).
- `SEC-014` `[C]` **Admin has full control** over what each role may use: switch features and filters on or off, and change a role's capabilities and scope after the role or user was created. Changes apply to existing users of that role.

### 10.3 What the UI does with scope

- Principal buttons show only allowed principals; with exactly one allowed, it is preselected and the control is a static label.
- Region, branch and similar filters list only allowed values; with exactly one allowed, preselected.
- A playlist or layout shared across users still renders per viewer scope; a dashboard the viewer is not allowed to see is skipped in rotation.
- Kiosk displays use a display credential that is itself a user-like identity with its own role and scope (DEC-012).

## 11. Authentication: admin-created local accounts (DEC-007)

**Owner decision `[C]`:** there is no Microsoft administrator, so **Microsoft sign-in is skipped for now.** Release 1 uses **username and password accounts created by an admin**; users can change their own password after login. Authentication sits behind an `AuthProvider` interface, so Microsoft (or another provider) can be added later without changing roles, scopes or the rest of the app.

```mermaid
flowchart TD
  A[Admin creates user<br/>name, username, roles, scope] --> P[Admin sets a temporary password]
  P --> I[Admin gives the user the access info<br/>site address, username, temporary password]
  I --> S[User signs in with username + password]
  S --> V{"Enabled user<br/>and password correct?"}
  V -- no --> F[Generic error; failures counted;<br/>temporary lockout after repeated failures]
  V -- yes --> CH{"Temporary password?"}
  CH -- yes --> FC[Forced password change]
  CH -- no --> OK[Session with roles and scope]
  FC --> OK
  OK --> C2[User can change password any time]
```

- `AUTH-001` `[C]` Only users created by an admin can sign in; nothing is created automatically.
- `AUTH-003` `[C]` Local accounts: username and password. The user can change their password after login.
- `AUTH-004` `[P]` Password rules: hashed with Argon2id; admin-set password is temporary and must be changed at first login; minimum length (12 proposed) and a check against common or breached passwords; rate limiting and temporary lockout after repeated failures (5 proposed); the password is shown to the admin once and never stored or logged in clear text.
- `AUTH-005` `[P]` **Password reset is done by an admin** (new temporary password, forced change at next login), because no email service is assumed. Self-service reset by email is a later option.
- `AUTH-006` `[C]` After the admin sets the role, the admin screen shows an **onboarding information** block to copy: site address, username, and the temporary password (once).
- `AUTH-007` `[P]` Sessions use HttpOnly, Secure, SameSite cookies over HTTPS, CSRF protection, idle and absolute timeouts and sign-out; every sign-in, failure, lockout, password change and reset is audited without secrets.
- `AUTH-008` `[P]` Kiosk displays use their own revocable credential (DEC-012), not a person's account.
- `AUTH-009` `[D]` **Deferred:** Microsoft sign-in (Entra ID, OpenID Connect) needs a Microsoft administrator to register the app. When available, add a provider that links a user by tenant id and object id, still only for admin-created users. The `auth_identities` table is reserved for that.
- `AUTH-010` `[C]` **MFA is configurable per user.** Admin opens a user and has an **Enable / Disable MFA** button. Only users who need MFA have it. When enabled, the user enrolls at the next sign-in (authenticator-app QR or key) and then enters a code after the password every time.
- `AUTH-011` `[C]` MFA method: **TOTP** (authenticator app, RFC 6238: 6 digits, 30-second step, one step of clock drift allowed, a used code cannot be reused). Recovery codes shown once at enrollment; the admin can reset a user's MFA (lost phone) or disable it; users cannot disable their own MFA; MFA secrets are encrypted at rest; code attempts are rate-limited; enable, disable, enroll and reset are audited. Capability: `mfa.manage` (assignable per role). Other methods (for example passkeys) can be added later.

## 12. AI service boundary

The AI chat (`07-ai-chat.md`) is a server module: `/api/ai/chat` builds the user's AccessContext, loads the **active provider profile** (settings, `07` section 9), calls the model through an `LlmClient` with the matching adapter (`openai-compatible`, `anthropic`, `gemini`), validates tool arguments, runs the read-only tools through the same services and repositories as the dashboards, runs exact arithmetic in a `calculate` tool, verifies every number in the answer, and returns structured result blocks. The model has no database access of any kind. Provider keys are stored encrypted in the database and decrypted only inside the server when a call is made; the master key comes from server secrets (DEC-049). A daily job deletes chat data older than 60 days. Endpoints: `POST /api/ai/chat`, `GET /api/ai/sessions`, `GET /api/ai/results/:resultId`, `POST /api/ai/results/:resultId/refresh`.

## 13. Data refresh and auto-update (DEC-046, DEC-048)

**Owner facts `[C]`:** an ETL loads data from the source systems, refreshes materialized views and runs stored procedures. A run takes **almost 3 hours** (nightly today). Dashboards must **update automatically** when data changes.

**Design `[P]`:** react to the **end of a completed ETL run**, not to every row change. During a 3-hour run tables change constantly and may be half-updated, so row-level change detection would show inconsistent numbers and put load on the database.

```mermaid
sequenceDiagram
  participant ETL as ETL job
  participant DB as PostgreSQL
  participant API as API server
  participant UI as Browsers (dashboards, kiosks, AI results)
  ETL->>DB: load tables, refresh materialized views, run procedures
  ETL->>DB: refresh log row is written (table already exists)
  API->>DB: poll the refresh log (default every 60 s, NOTIFY optional later)
  API->>API: bump dataVersion, set dataAsOf, clear caches
  API-->>UI: server-sent event "data refreshed"
  UI->>API: refetch current queries (cache invalidated)
  UI->>UI: show updated data and new as-of time
```

- `OPS-004` `[C]` Dashboards, kiosk/playlist displays and open AI results update automatically after a refresh finishes, without a manual reload and without flicker (current view stays until new data arrives).
- `OPS-005` `[C]` The ETL refresh log is **`dwh_prod.etl_refresh_log`** with columns `process_name`, `start_time`, `end_time`, `status`, `message` (owner, DEC-048). Newest row when this was written: process `Refresh Materialized Views`, start 2026-10-22 04:24:28.270, end 2026-10-22 05:14:07.327, status `SUCCESS`, empty message. The app only reads it. `dataAsOf` is the `end_time` of the newest row whose `process_name` is the configured process and whose `status` is the configured success value; a change in that value is the "data refreshed" signal. The API polls it (default every 60 seconds, configurable). Table, column names, process name and success value are configuration, not code. Open: whether other processes must also finish before data counts as ready (the whole ETL takes about 3 hours; the logged materialized-view step about 50 minutes), the `status` values while running or failed, and the time zone of the timestamps (DEC-077).
- `OPS-006` `[P]` `dataVersion` is part of every cache key, so stale cached numbers are never served after a refresh. Cache lifetime is configuration.
- `OPS-007` `[P]` While a run is in progress the UI may show "refresh in progress, data as of <previous completion>" (needs the log to record the start). Failed runs show the previous completed `dataAsOf` and a warning.
- `OPS-008` `[P]` **Avoid read blocking:** a plain `REFRESH MATERIALIZED VIEW` locks readers for its duration. Prefer `REFRESH MATERIALIZED VIEW CONCURRENTLY` (needs a unique index on each view) so dashboards stay readable during the run. This is a database/ETL decision for the owner.
- `OPS-009` `[P]` Data findings checks (`08-data-findings.md`) run after each completed refresh instead of on a fixed schedule.
- `OPS-010` `[P]` A reopened AI result older than the current `dataVersion` shows "newer data available" with a Refresh action (`07` section 7).
- Near-real-time later: the same event path works if refresh becomes incremental; the signal moves from "run finished" to "batch finished".
- `OPS-011` `[P]` **Freshness tiers are configuration per data source**, so each source can move at its own pace and the UI and AI wording follow it:

```mermaid
flowchart LR
  T1["Tier 1: monthly data,<br/>refreshed by the nightly ETL (now)"] --> T2["Tier 2: day-level rows, refreshed<br/>nightly (SE sell-out, planned)"]
  T2 --> T3["Tier 3: near real time<br/>(more frequent ETL or snapshots, later)"]
```

Each registered data source declares its grain (`month`, `day`, `intraday`) and its refresh signal; `dataAsOf`, caching and the AI's "month to date" behavior are derived from that declaration.

## 14. Database connection and safety (DEC-009)

**Owner decision `[C]`:** use the **existing PostgreSQL**; no second database (the database is a staging copy loaded by the ETL from production, and a second copy would have to be mirrored again). Connection comes from `.env` (`.env.example` is in the repository): `PG_HOST`, `PG_PORT`, `PG_DB`, `PG_USER`, `PG_PASSWORD`. The owner uses one full-access user for now.

- `OPS-012` `[C]` **App data lives in its own schema `app_config` in the same database**, never inside `dwh_prod`. Migrations create and change only that schema; a startup check and a test refuse any migration that touches another schema. The ETL must never drop, truncate or recreate `app_config` (DEC-080).
- `OPS-013` `[C]` **Single user is supported.** With only `PG_USER` set, the app uses two connection pools with the same credentials: a **business pool** whose sessions are opened read-only (`default_transaction_read_only = on`, statement timeout) used for every dashboard, pivot, AI-tool and export query; and an **app pool** that writes only to `app_config`. Business queries are fixed, parameterized and built from the column registry; no code path runs DDL or writes against `dwh_prod`.
- `OPS-014` `[P]` **Optional later:** set `PG_RO_USER` / `PG_RO_PASSWORD` to use a separate read-only database user for the business pool (same database, no mirroring, five minutes of setup). It is not required, but it is the one control that still protects the warehouse if the app has a bug or is compromised, because read-only sessions are enforced by the app, while a read-only user is enforced by PostgreSQL.
- `OPS-015` `[P]` **Backups.** The staging database can be rebuilt from production, but `app_config` (users, roles, layouts, playlists, saved views, settings) cannot. Include the `app_config` schema in a regular backup stored off the database server (for example a nightly schema-only dump plus data), and test a restore before go-live. The encryption master key is backed up separately (`SEC-015`).
- `OPS-016` `[P]` ETL interplay: dashboards read the materialized views only after the refresh log shows success (`OPS-005`); the app never locks or refreshes views itself.
- `OPS-017` `[C]` **The ETL sometimes drops, truncates and adds columns** (owner, DEC-084). Protection of `app_config`, in layers: (1) its own schema, never `dwh_prod`; (2) the ETL owner lists exactly which schemas and objects it drops or truncates (DEC-080); (3) if the ETL runs as a different database user, that user gets no privileges on `app_config`; (4) if it runs as the same user, optional guards (needs a superuser once): an event trigger that refuses `DROP` of `app_config` objects and `BEFORE TRUNCATE` triggers on app tables, both supported by PostgreSQL 12; (5) the off-server backup (`OPS-015`); (6) a startup self-check refuses to run if `app_config` or its migration table is missing or changed, and prints the restore steps.
- `OPS-018` `[P]` **Last-good results.** Dashboards, pivots, AI tools and exports serve the **last good result** whenever the source is not ready: during the ETL load (tables are truncated and reloaded; only the final materialized-view refresh is logged), while a materialized view refreshes (a plain refresh blocks readers for up to about 50 minutes), or when an object is missing. The UI shows "Data update in progress, showing data as of <last good time>". A dashboard is never blank and never shows partial numbers. Last-good results are kept per query and per `dataVersion` in an `app_config` cache table so they survive an app restart, with a size cap.
- `OPS-019` `[P]` **Maintenance window.** The expected ETL window (for example from the start of loading to the end of the materialized-view refresh) is configuration, learned from the log history by default. Reads inside it go to last-good. Asking the ETL to log the start and end of the load step as well would make this exact instead of estimated (DEC-082).
- `OPS-020` `[P]` **Partial-data guard.** If a fresh result is implausibly empty or far smaller than the last good result (for example the current month vanishes), treat it as incomplete: serve last-good, show the warning, and raise a data finding (`08-data-findings.md`).
- `OPS-021` `[P]` **Schema drift.** Queries use explicit column lists, never `SELECT *`, so added columns change nothing. A drift check runs at startup and after every refresh: it compares the column registry with `information_schema`; an added column is ignored; a missing, renamed or retyped column raises a `SCHEMA_DRIFT` finding and disables only the affected widgets. Dropped-and-recreated objects are retried with back-off.

## 15. Server and runtime (DEC-008)

**Owner facts `[C]`:** the server runs Ubuntu 20.04 with Docker; Node.js v24.21.0; PostgreSQL 12.22.

- `OPS-022` `[C]` **The app runs in Docker**, not on the host: a Node 24 base image pinned in the Dockerfile (`engines` and `.nvmrc` also say 24), `docker-compose.yml` with a restart policy, a health check (`/api/health`), configuration from `.env`, and a reverse proxy for HTTPS (for example Caddy or nginx in a container, `[P]`). The host operating system therefore matters little. Node 24 is a supported LTS line; check the Node.js release schedule when Phase 1 starts (at the time of writing it was entering its maintenance phase and a newer LTS line was due), and plan a review of the Node version around 2027.
- `OPS-023` `[C]` **PostgreSQL 12 compatibility.** Every query and migration must work on PostgreSQL 12.22. Rules: IDs are generated in the app (UUID v7), not by `gen_random_uuid()` (built in only from PostgreSQL 13; on 12 it needs the `pgcrypto` extension); no `MERGE` (PostgreSQL 15), no `UNIQUE NULLS NOT DISTINCT` (15), no newer JSON functions; `ROLLUP`, `GROUPING SETS`, generated columns, `numeric`, `REFRESH MATERIALIZED VIEW CONCURRENTLY` and event triggers are available. CI runs the database tests against a PostgreSQL 12 container **and** a current PostgreSQL, so a later upgrade does not break the app.
- `OPS-024` `[P]` **Support status to plan for (outside this project):** Ubuntu 20.04 left standard support in 2025 (security updates only through paid extended maintenance), and PostgreSQL 12 reached end of life in November 2024 (12.22 was its final release). Because the app is containerised and tested on 12 and a current version, upgrading the host and the database later is possible without changing the app. The warehouse's security patching is the owner's decision.
- `OPS-025` `[P]` Time zone: containers run with `TZ` set to the business time zone (DEC-006); timestamps are stored and compared consistently with the ETL log's type (DEC-081).
