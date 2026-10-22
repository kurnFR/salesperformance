# Data Model

Related: [`01-metrics-contract.md`](01-metrics-contract.md) · [`02-architecture.md`](02-architecture.md)

## 1. What this document does and does not contain

| Category | In this document? | Why |
|---|---|---|
| **App-owned data structures** (layout, rules, playlist, DTOs) | **Yes, fully specified** | We design them; they define the contract between UI, API and storage |
| **App-owned config table names** | **Yes, as proposals** under schema `app_config` | Need owner approval before any migration is applied |
| **Business/source table and column names** | **Yes, as supplied by the owner** (data dictionary `table_name`, Tableau workbook) | Table names are owner-supplied. Column names come from the Tableau CSV connections and are **unverified against PostgreSQL** until checked with `information_schema.columns` (`DATA-003`) |
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
{ "principalId": "INA", "startMonth": "2026-10", "endMonth": "2026-10", "regionId": null, "branchId": null, "timezone": "TBD" }
```

### 2.5 Access policy (role, capabilities, scope) (`[P]`)

```json
{
  "schemaVersion": 1,
  "roleId": "role-asw-west",
  "name": "ASW regional viewer",
  "capabilities": ["dashboard.view:sales-performance", "ai.chat.use", "findings.view"],
  "filters": {
    "principal": { "visibility": "editable" },
    "region":    { "visibility": "editable" },
    "branch":    { "visibility": "hidden" }
  },
  "scope": {
    "principal": { "mode": "list", "values": ["ASW"] },
    "region":    { "mode": "list", "values": ["TBD-region-key-1", "TBD-region-key-2"] },
    "dashboard": { "mode": "all" }
  }
}
```

Full-access example: `"principal": { "mode": "all" }, "region": { "mode": "all" }`. Filter `visibility` is `hidden`, `locked` or `editable`. Capability names are validated against the code's capability catalog; scope values are validated against known data values on save. User assignment: `{ "userId": "...", "roleIds": ["..."], "overrides": { "region": { "add": [], "remove": [] } } }`.

### 2.6 Principal registry (`[P]`)

Maps each principal to its physical tables so no query code hard-codes `ina`/`asw` names. Table names are allowlisted and come from the owner-approved data dictionary.

```json
{
  "principals": [
    { "id": "INA", "label": "INA", "tables": {
        "performance": "dwh_prod.v_tsellinselloutina",
        "mapping": "dwh_prod.v_sr_per_branchina",
        "masterDistributor": "dwh_prod.vt_sr_per_rsmina",
        "activeDistributors": { "table": "dwh_prod.distaktif", "principalFilter": "regionCodesFromMapping" },
        "dailyUpload": "dwh_prod.mt_daily_updateina",
        "dailyStockUpload": "dwh_prod.mt_daily_updatestockina" } },
    { "id": "ASW", "label": "ASW", "tables": {
        "performance": "dwh_prod.v_tsellinselloutasw",
        "mapping": "dwh_prod.v_sr_per_branch",
        "masterDistributor": "dwh_prod.vt_sr_per_rsmasw",
        "activeDistributors": { "table": "dwh_prod.distaktif", "principalFilter": "regionCodesFromMapping" },
        "dailyUpload": "dwh_prod.mt_daily_updateasw",
        "dailyStockUpload": "dwh_prod.mt_daily_updatestockasw" } }
  ],
  "sharedTables": { "dateDimension": "dwh_prod.dimdate" }
}
```

The ASW mapping table is `dwh_prod.v_sr_per_branch` (confirmed by the owner: a name without the `ina` suffix is ASW; each principal has its own region, branch and other data). `distaktif` currently holds **both** INA and ASW regions (owner, DEC-044) and will be split into separate tables later. Until then each principal reads it with a filter on that principal's region codes taken from its mapping table; after the split, only the registry entry changes (`table` becomes the principal's own table and `principalFilter` becomes `none`). Principal-specific row filters (for example the `distaktif` table is shared and filtered by region names) must be defined per approved mapping.

### 2.7 Tooltip-ready metric DTOs (`[P]`, supports DASH-001)

Every metric point carries what its tooltip needs, so the UI never recalculates:

```json
{
  "principalId": "INA",
  "regionId": "TBD-region-key",
  "regionLabel": "INA JAWA 1",
  "month": "2026-10",
  "sellIn":  { "target": 0, "actual": 0, "achievementPct": 0.0, "status": "red" },
  "sellOut": { "target": 0, "actual": 0, "achievementPct": 0.0, "status": "red" }
}
```

Growth point (Growth Sell In / Sell Out):

```json
{
  "regionId": "TBD-region-key",
  "month": "2026-09",
  "thisYear": { "target": 0, "actual": 0, "achievementPct": 0.0, "status": "green" },
  "lastYear": { "target": 0, "actual": 0, "achievementPct": 0.0, "status": "red" },
  "growthPct": 0.0,
  "growthLabelVisible": true
}
```

All numbers above are placeholders. `growthPct` is `null` (not 0) when last year's actual is zero or missing. The window request is `GET .../growth?month=2026-10&window=7` and the API returns the 7 months ending at `month`, each with last-year counterpart.

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
| `app_config.roles` | `id`, `name`, `description`, `is_system`, `created_at` | Admin-managed; seed roles are normal rows |
| `app_config.role_capabilities` | `role_id`, `capability` | Capability validated against the code catalog |
| `app_config.role_scopes` | `role_id`, `dimension`, `mode` (`all`/`list`), `values_json` | One row per dimension per role |
| `app_config.user_roles` | `user_id`, `role_id` | `user_id` from the identity provider (DEC-007) |
| `app_config.user_scope_overrides` | `user_id`, `dimension`, `add_json`, `remove_json` | Optional |
| `app_config.principals` | `id`, `label`, `tables_json`, `enabled` | Principal registry (2.6) |
| `app_config.users` | `id`, `display_name`, `sign_in_method` (`local` now; `microsoft` later), `enabled`, `created_by`, `created_at` | Created by admins only |
| `app_config.auth_identities` | `user_id`, `provider`, `tenant_id`, `object_id`, `account_hint`, `linked_at` | **Reserved for deferred Microsoft sign-in** (not used in release 1) |
| `app_config.local_credentials` | `user_id`, `username`, `password_hash`, `must_change`, `failed_attempts`, `locked_until`, `changed_at` | Argon2id hash only |
| `app_config.data_findings`, `data_finding_notes` | See `08-data-findings.md` section 3.1 | Findings and history |
| `app_config.ai_sessions`, `ai_messages`, `ai_results` | `user_id`, question, tool-call summary, result spec JSON, `data_as_of`, timestamps | Per-user chat history and reopenable results; **deleted after 60 days** by a daily job |
| `app_config.audit_events` | `id`, `actor`, `action`, `target`, `summary_json`, `created_at` | Safe summaries only, no secrets; includes permission changes |

Requirements if approved: explicit migrations with constraints, indexes and rollback notes; keep revision history for shared layouts and rules; validate referenced dashboards exist before publishing; never silently overwrite a newer revision.

## 4. Source tables and mapping

### 4.1 Tables supplied by the owner (data dictionary `table_name`)

Source of table names: the repository file `table_name`. Column lists come from the Tableau workbook's CSV connections (INA). `[R]` = reference-derived, not yet verified in PostgreSQL.

| Logical name | INA table | ASW table | Role in the product |
|---|---|---|---|
| Performance (sales, targets) | `dwh_prod.v_tsellinselloutina` | `dwh_prod.v_tsellinselloutasw` | Sell In / Sell Out actual and target per selling point and month |
| Mapping (org + salesman) | `dwh_prod.v_sr_per_branchina` | `dwh_prod.v_sr_per_branch` (confirmed: no suffix means ASW) | Region, branch, area, selling point, salesman (SE) |
| Master distributor | `dwh_prod.vt_sr_per_rsmina` | `dwh_prod.vt_sr_per_rsmasw` | Distributor hierarchy; join target for upload data |
| Date dimension | `dwh_prod.dimdate` | `dwh_prod.dimdate` (shared) | Calendar and working days |
| Active distributors | `dwh_prod.distaktif` (shared today) | same table, filtered by ASW region codes; to be split later (DEC-044) | Total / resigned / active distributors per region and month. **Uploaded manually from Excel; may be stale or contain errors** (see `08-data-findings.md`) |
| Daily upload | `dwh_prod.mt_daily_updateina` | `dwh_prod.mt_daily_updateasw` | Daily upload log per selling point |
| Daily stock upload | `dwh_prod.mt_daily_updatestockina` | `dwh_prod.mt_daily_updatestockasw` | Daily stock upload log |

### 4.2 Columns (`[R]`, verify in PostgreSQL)

- **Performance:** `SELLINGPOINTCODE`, `SELLINGPOINTNAME`, `BRANCHCODE`, `BRANCHNAME`, `REGIONCODE`, `REGIONNAME`, `KODEAREA`, `NAMAAREA`, `KOTA`, `Tahun`, `Bulan`, `Periode` (month date), `TargetQtyEskaSLO`, `TargetValueEskaSLO`, `RealQtySLI`, `RealQtySLO`, `RealSellInReg`, `RealSellInFes`, **`TOTALRealSellIn`**, `RealSellOutReg`, `RealSellOutFes`, **`TOTALRealSellOut`**, `TARGETSellInReg`, `TARGETSellInFes`, **`TOTALTargetSellIn`**, `TARGETSellOutReg`, `TARGETSellOutFes`, **`TOTALTargetSellOut`**. Bold = fields used by the four reference sheets.
- **Mapping:** `KOTA`, `REGIONCODE`, `REGIONNAME`, `BRANCHCODE`, `BRANCHNAME`, `KODEAREA`, `NAMAAREA`, `KODESCABANG`, `SELLINGPOINTNAME`, `SLSNO`, `SLSNAME`, `CUSTNO`, `CUSTNAME`, `SLSTP`, `TEAM`, `TEAMNAME`, `BranchActiveFlag`, `FLAG_AKTIF`, `NPWP`, `Source`.
- **Master distributor:** `KODESCABANG`, `BRANCHCODE`, `BRANCHNAME`, `REGIONCODE`, `REGIONNAME`, `KODEAREA`, `NAMAAREA`, `KOTA`, `SELLINGPOINTNAME`.
- **Date dimension:** `date_dim_id`, `dates`, `daysofyears`, `months`, `years`, `quarters`, `monthsname`, `daysofmonths`, `daysofweekname`, `weeknumber`, `workingday(5)`, `workingday(6)`, `daysofweek`, `ganjilgenap`, `holiday`, **`networkeddays`**, `monthyear`.
- **Active distributors:** `RegionCode`, `RegionName`, `AreaCode`, `AreaName`, `Bulan`, `TOTAL DIST`, `Total Resign`, `Total Aktif`.
- **Daily upload / stock upload:** `SELLINGPOINTCODE`, `SELLINGPOINTNAME`, `INVOICE_DATE` (stock: `STOCKUPDATEDATE`), `KOTA`, `Updated`.

Postgres folds unquoted identifiers to lowercase and column names with spaces or parentheses (`TOTAL DIST`, `workingday(5)`) may differ from the CSV headers. Never copy these names into SQL without checking.

### 4.3 Joins observed in the workbook (`[R]`)

- Master distributor `KOTA` = Daily upload `KOTA`.
- Master distributor `KOTA` = Daily stock upload `KOTA`.
- All other tables are used as separate sources blended on region, branch, area, selling point and month in Tableau; the exact keys must be defined by the owner (DEC-001).

### 4.4 Organisation hierarchy and role mapping (`[R]`, confirm in DEC-003)

| Role | Source level | Evidence |
|---|---|---|
| RSM | `REGIONNAME` (5 regions give 5 RSM) | Region-level achievement flags |
| ASM | `BRANCHNAME` (19) | Branch-level flags |
| SPV | `NAMAAREA` (75) | Area-level flags |
| DIST | `SELLINGPOINTNAME` (144) | Selling-point flags |
| SE | `SLSNO` distinct (260), with codes containing "SOA" or "SOI" excluded in some counts | `SE Excl SOA/SOI` fields |

Hierarchy: Region (RSM) > Branch (ASM) > Area (SPV) > Selling point (DIST) > Salesman (SE).

### 4.5 Mapping status per dashboard item

| Item | Source (table, columns) | Grain | Status |
|---|---|---|---|
| Sell IN actual and target | Performance: `TOTALRealSellIn`, `TOTALTargetSellIn` | Selling point per month | Derived from Tableau; DB verification pending |
| Sell Out actual and target | Performance: `TOTALRealSellOut`, `TOTALTargetSellOut` | Selling point per month | Derived from Tableau; DB verification pending |
| Region list and order | Performance / Mapping: `REGIONCODE`, `REGIONNAME` | Region | Display order undefined (DEC-001) |
| Team totals RSM/ASM/SPV/SE | Mapping: region, branch, area, `SLSNO` | Salesman | Derived; active flags undefined (DEC-028) |
| Role achievement counts | Performance aggregated per entity per month; achieved when achievement >= working-day progress | Entity per month | Rule confirmed (DEC-021); DB verification pending |
| Working-day progress | `dimdate`: `dates`, `networkeddays`, `monthyear`; summed per month, no custom calendar | Date | Confirmed approach (DEC-027); verify 18 / 27 on 22-Oct-2026 |
| Growth Sell In / Out | Performance, same month last year (12 months earlier) | Region per month | Derived from Tableau |
| Active distributors per region | `distaktif`: `Total Aktif`, `TOTAL DIST`, `Bulan`; % = active / total | Region/area per month | Formula confirmed (DEC-024); DB verification pending |
| Daily upload status | Daily upload: `INVOICE_DATE`, `Updated`, `KOTA`; join Master distributor on `KOTA` | Selling point per day | Derived; stock variant open (DEC-026) |

An item is ready for production use only when its row is `Approved` and has a verified example (definition of ready in `05-roadmap-and-testing.md`). Until then it uses mock fixtures.

### 4.6 Verification queries to run (owner or analyst)

1. `SELECT column_name, data_type FROM information_schema.columns WHERE table_schema='dwh_prod' AND table_name IN (...)` for every table above.
2. Row counts and date ranges per table; confirm the grain of the performance view (one row per selling point per month?).
3. Distinct `REGIONNAME` per principal (expect INA: JAWA 1, JAWA 2, PULAU 1, SUMATERA 1, SUMATERA 2; Tableau lists ASW with six regions including SUMATERA 3).
4. For Oct 2026: `networkeddays` per date, to reproduce 66.67% on 22-Oct-2026.
5. Independent hand calculation of one region's Sell IN % for one month.

## 5. Mock fixtures

- Live in `fixtures/`, every file named or headed `MOCK`; never used in production paths and never treated as business evidence.
- Golden test cases (`fixtures/golden/`) cover: 0/0, 0/5, 1/5 = 20%, 5/5, 3 of 5 working days = 60%, the color-rule table in `01-metrics-contract.md`.
- A dev-only synthetic Postgres (docker-compose) may be added for end-to-end testing; it must live in a clearly named dev-only schema and never be deployed.
