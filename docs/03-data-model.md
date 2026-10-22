# Data Model

Related: [`01-metrics-contract.md`](01-metrics-contract.md) · [`02-architecture.md`](02-architecture.md)

## 1. What this document does and does not contain

| Category | In this document? | Why |
|---|---|---|
| **App-owned data structures** (layout, rules, playlist, DTOs) | **Yes, fully specified** | We design them; they define the contract between UI, API and storage |
| **App-owned config table names** | **Yes, as proposals** under schema `app_config` | Need owner approval before any migration is applied |
| **Business/source table and column names** | **No. Template only** | Unknown until the owner supplies the schema; guessing is forbidden (`DATA-003`) |
| **Flowcharts** | Yes (Mermaid) | See `01` and `02` |

## 2. App-owned JSON structures (`[P]`)

All carry `schemaVersion` so they can be migrated. All are validated with Zod on read and write. They contain no executable code and no raw SQL.

### 2.1 Dashboard layout

```json
{
  "schemaVersion": 1,
  "dashboardId": "sales-performance",
  "version": 7,
  "grid": { "columns": 12, "rowHeight": 24, "gap": 8 },
  "background": { "assetId": "asset-001", "fit": "cover", "position": "center", "overlayOpacity": 0.4 },
  "widgets": [
    {
      "id": "w-kpi-rsm",
      "type": "kpiCard",
      "title": "RSM",
      "binding": "salesPerformance.roleCard.RSM",
      "options": { "showIn": true, "showOut": true },
      "position": { "x": 0, "y": 0, "w": 3, "h": 4 },
      "visible": true
    },
    {
      "id": "w-text-1",
      "type": "text",
      "content": [ { "kind": "heading", "text": "Sales Performance" } ],
      "position": { "x": 0, "y": 4, "w": 12, "h": 2 }
    }
  ]
}
```

- `type` is from an allowlisted widget registry: `kpiCard`, `barChart`, `workingDayProgress`, `text`, `image`.
- `binding` is an **allowlisted key** resolved server-side, never a table name, column or SQL.
- Text content is structured nodes with limited formatting, never raw HTML.

### 2.2 Achievement rule set

```json
{
  "schemaVersion": 1,
  "version": 3,
  "warningMin": 70,
  "successMin": 100,
  "comparison": { "workingDayProgress": ">=" },
  "bands": { "belowWarning": "danger", "warning": "warning", "success": "success", "onTrack": "success" },
  "colors": { "success": "token:success", "warning": "token:warning", "danger": "token:danger", "unavailable": "token:muted" },
  "evaluationOrder": ["workingDayPace", "achievementBand"]
}
```

Validation: `0 <= warningMin < successMin`, colors from supported tokens meeting contrast, `evaluationOrder` fixed to the confirmed rule in `01-metrics-contract.md` section 4.

### 2.3 Playlist

```json
{
  "schemaVersion": 1,
  "id": "playlist-main",
  "name": "Main wall display",
  "enabled": true,
  "defaultDurationSeconds": 10,
  "onLoadFailure": "skip",
  "items": [
    { "dashboardId": "sales-performance", "enabled": true, "durationSeconds": 10, "order": 1 }
  ]
}
```

Validation: `durationSeconds >= 5` (`[P]` minimum), referenced dashboards must exist, `onLoadFailure` is `skip` or `keepLast`.

### 2.4 Normalized filter object

```json
{ "startDate": "2026-10-01", "endDate": "2026-10-31", "regionId": null, "timezone": "TBD" }
```

## 3. Proposed `app_config` tables (not approved; no migration until owner signs off, DEC-009)

Separate schema keeps app-owned data apart from read-only business tables. Names are proposals only.

| Table | Key columns | Notes |
|---|---|---|
| `app_config.dashboards` | `id`, `route_key`, `name`, `description`, `visibility`, `published_version`, `created_at` | One row per dashboard |
| `app_config.dashboard_layout_versions` | `dashboard_id`, `version`, `layout_json`, `schema_version`, `created_by`, `created_at` | Append-only; unique `(dashboard_id, version)` for optimistic concurrency |
| `app_config.achievement_rule_sets` | `version`, `rules_json`, `is_active`, `created_by`, `created_at` | Exactly one active |
| `app_config.playlists` | `id`, `name`, `enabled`, `default_duration_seconds`, `on_load_failure` | |
| `app_config.playlist_items` | `playlist_id`, `dashboard_id`, `position`, `duration_seconds`, `enabled` | FK to dashboards |
| `app_config.assets` | `id`, `storage_key`, `mime_type`, `size_bytes`, `width`, `height`, `created_by` | Approved storage only |
| `app_config.audit_events` | `id`, `actor`, `action`, `target`, `summary_json`, `created_at` | Safe summaries only, no secrets |

Requirements if approved: explicit migrations with constraints, indexes and rollback notes; keep revision history for shared layouts and rules; validate referenced dashboards exist before publishing; never silently overwrite a newer revision.

## 4. Source mapping template

Owner (or analyst) fills this in **per dashboard item** before Phase 2 connects to PostgreSQL. Until a row is `Approved`, the item uses mock fixtures only. An item is "ready" when the row is complete and has a verified example (see definition of ready in `05-roadmap-and-testing.md`).

| Item | Source table/view | Columns | Grain | Join keys | Filters / exclusions | Null/zero policy | Verified example | Status |
|---|---|---|---|---|---|---|---|---|
| Role headcount (RSM/ASM/SPV/SE/DIST) | TBD | TBD | TBD | TBD | TBD | TBD | TBD | Awaiting owner |
| IN achievement per person | TBD | TBD | TBD | TBD | TBD | TBD | TBD | Awaiting owner |
| OUT achievement per person | TBD | TBD | TBD | TBD | TBD | TBD | TBD | Awaiting owner |
| Sell IN actual and target | TBD | TBD | TBD | TBD | TBD | TBD | TBD | Awaiting owner |
| Sell Out actual and target | TBD | TBD | TBD | TBD | TBD | TBD | TBD | Awaiting owner |
| Regions (id, label, order, hierarchy) | TBD | TBD | TBD | TBD | TBD | TBD | TBD | Awaiting owner |
| Org hierarchy / target roll-up (RSM > ASM > SPV > SE) | TBD | TBD | TBD | TBD | TBD | TBD | TBD | Awaiting owner (DEC-003) |
| Working-day calendar | TBD | TBD | TBD | TBD | TBD | TBD | TBD | Awaiting owner |

**Fastest way to fill this in:** export the existing Tableau workbook (`.twbx` / `.twb`) into `docs/reference/`. Its calculated fields and data-source queries already contain the real formulas, joins and filters.

## 5. Mock fixtures

- Live in `fixtures/`, every file named or headed `MOCK`; never used in production paths and never treated as business evidence.
- Golden test cases (`fixtures/golden/`) cover: 0/0, 0/5, 1/5 = 20%, 5/5, 3 of 5 working days = 60%, the color-rule table in `01-metrics-contract.md`.
- A dev-only synthetic Postgres (docker-compose) may be added for end-to-end testing; it must live in a clearly named dev-only schema and never be deployed.
