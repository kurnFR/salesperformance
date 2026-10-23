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
  "displayId": "display-ceo-tv",
  "defaultPresentation": "half",
  "defaultDurationSeconds": 10,
  "onLoadFailure": "skip",
  "items": [
    { "dashboardId": "sales-performance", "enabled": true, "durationSeconds": 10, "presentation": "half", "order": 1 }
  ]
}
```

`presentation` is an advisory hint (`half`, `full` or `auto`); the layout always follows the actual window. Validation: `durationSeconds >= 5` (`[P]` minimum), referenced dashboards must exist, `onLoadFailure` is `skip` or `keepLast`.

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
  },
  "departmentId": "sales"
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
  "sharedTables": {
    "dateDimension": "dwh_prod.dimdate",
    "etlRefreshLog": { "table": "dwh_prod.etl_refresh_log", "processName": "Refresh Materialized Views", "startColumn": "start_time", "finishedColumn": "end_time", "statusColumn": "status", "successValue": "SUCCESS", "messageColumn": "message" }
  }
}
```

The ASW mapping table is `dwh_prod.v_sr_per_branch` (confirmed by the owner: a name without the `ina` suffix is ASW; each principal has its own region, branch and other data). `distaktif` currently holds **both** INA and ASW regions (owner, DEC-044) and will be split into separate tables later. Until then each principal reads it with a filter on that principal's region codes taken from its mapping table; after the split, only the registry entry changes (`table` becomes the principal's own table and `principalFilter` becomes `none`). Principal-specific row filters (for example the `distaktif` table is shared and filtered by region names) must be defined per approved mapping.

### 2.7 AI provider profile (`[P]`)

```json
{
  "id": "prov-001",
  "name": "Local gateway",
  "adapter": "openai-compatible",
  "baseUrl": "TBD (for example the local 9Router or OmniRoute address)",
  "model": "TBD (typed or picked from the provider's list)",
  "apiKey": { "stored": true, "last4": "TBD" },
  "enabled": true,
  "isActive": true,
  "qualification": { "status": "passed", "testedAt": "TBD" },
  "dataSharingAcknowledgedBy": "TBD",
  "createdBy": "TBD"
}
```

The API key is **never** returned: the API exposes only `stored` and `last4`. Adapter is one of `openai-compatible`, `anthropic`, `gemini`.

### 2.8 Sales Detail field catalog and query (`[P]`, `PIVOT-011`)

The pivot never accepts a column name from the browser. It accepts **ids** from this allowlisted catalog; the server maps ids to approved columns.

```json
{
  "schemaVersion": 1,
  "dimensions": [
    { "id": "region",      "label": "Region",      "source": "REGIONNAME" },
    { "id": "branch",      "label": "Branch",      "source": "BRANCHNAME" },
    { "id": "spv",         "label": "SPV",         "source": "NAMAAREA" },
    { "id": "distributor", "label": "Distributor", "source": "SELLINGPOINTNAME" },
    { "id": "se",          "label": "Salesman (SE)", "source": "SLSNO / SLSNAME", "entitiesOnly": true },
    { "id": "customer",    "label": "Customer",    "source": "CUSTNO / CUSTNAME", "entitiesOnly": true },
    { "id": "city",        "label": "City",        "source": "KOTA", "optional": true }
  ],
  "hierarchies": [
    { "id": "salesOrg", "levels": [
      { "id": "region",      "dimension": "region",      "parent": null,          "measuresFrom": "performance" },
      { "id": "branch",      "dimension": "branch",      "parent": "region",      "measuresFrom": "performance" },
      { "id": "spv",         "dimension": "spv",         "parent": "branch",      "measuresFrom": "performance" },
      { "id": "distributor", "dimension": "distributor", "parent": "spv",         "measuresFrom": "performance" },
      { "id": "se",          "dimension": "se",          "parent": "distributor", "measuresFrom": "seMonthly", "entitiesFrom": "mapping", "status": "planned: SE target and sell-out arrive with the Marketing module (DEC-059, DEC-070)" },
      { "id": "customer",    "dimension": "customer",    "parent": "se",          "measuresFrom": null, "entitiesFrom": "mapping", "note": "customer code CUSTNO, name CUSTNAME; no customer-level facts (DEC-088)" }
    ] },
    { "id": "productCatalog", "department": "marketing", "status": "planned",
      "levels": [ { "id": "item", "label": "Product name", "key": "pcode", "measuresFrom": null } ],
      "note": "Product (item) is not part of the Sales organisation hierarchy; it belongs to the Marketing module, added later (DEC-060)" }
  ],
  "measures": [
    { "id": "targetIn",  "label": "Target Sell IN",  "source": "TOTALTargetSellIn" },
    { "id": "targetOut", "label": "Target Sell OUT", "source": "TOTALTargetSellOut" },
    { "id": "realIn",    "label": "Real Sell IN",    "source": "TOTALRealSellIn" },
    { "id": "realOut",   "label": "Real Sell OUT",   "source": "TOTALRealSellOut" },
    { "id": "targetInReg",  "label": "Target Sell IN (Regular)",  "source": "TARGETSellInReg",  "segment": "reg" },
    { "id": "targetInFes",  "label": "Target Sell IN (Festive)",  "source": "TARGETSellInFes",  "segment": "fes" },
    { "id": "realInReg",    "label": "Real Sell IN (Regular)",    "source": "RealSellInReg",    "segment": "reg" },
    { "id": "realInFes",    "label": "Real Sell IN (Festive)",    "source": "RealSellInFes",    "segment": "fes" },
    { "id": "targetOutReg", "label": "Target Sell OUT (Regular)", "source": "TARGETSellOutReg", "segment": "reg" },
    { "id": "targetOutFes", "label": "Target Sell OUT (Festive)", "source": "TARGETSellOutFes", "segment": "fes" },
    { "id": "realOutReg",   "label": "Real Sell OUT (Regular)",   "source": "RealSellOutReg",   "segment": "reg" },
    { "id": "realOutFes",   "label": "Real Sell OUT (Festive)",   "source": "RealSellOutFes",   "segment": "fes" },
    { "id": "qtyIn",        "label": "Real Sell IN (quantity)",    "source": "RealQtySLI",        "unit": "TBD (DEC-094)" },
    { "id": "qtyOut",       "label": "Real Sell OUT (quantity)",   "source": "RealQtySLO",        "unit": "TBD (DEC-094)" },
    { "id": "targetQtyOut", "label": "Target Sell OUT (quantity)", "source": "TargetQtyEskaSLO",  "unit": "TBD (DEC-094)" },
    { "id": "targetValueOutEska", "label": "Target Sell OUT (value, Eska)", "source": "TargetValueEskaSLO", "note": "compare with TOTALTargetSellOut (DEC-095)" },
    { "id": "seTarget",  "label": "SE target (month)",       "source": "TBD (DEC-070)", "level": "se", "status": "planned" },
    { "id": "seSellOut", "label": "SE sell out (month to date)", "source": "TBD (DEC-070)", "level": "se", "status": "planned" }
  ],
  "derived": [
    { "id": "achIn",  "label": "Achievement IN %",  "formula": "realIn / targetIn" },
    { "id": "achOut", "label": "Achievement OUT %", "formula": "realOut / targetOut" },
    { "id": "gapIn",  "label": "Shortfall IN",      "formula": "max(targetIn - realIn, 0)" },
    { "id": "paceIn", "label": "Pace gap IN",       "formula": "achIn - workingDayProgress" },
    { "id": "achQtyOut",      "label": "Sell OUT quantity achievement %", "formula": "qtyOut / targetQtyOut" },
    { "id": "valuePerUnitIn",  "label": "Sell IN value per unit",  "formula": "realIn / qtyIn" },
    { "id": "valuePerUnitOut", "label": "Sell OUT value per unit", "formula": "realOut / qtyOut" },
    { "id": "seAchOut", "label": "SE achievement %", "formula": "seSellOut / seTarget", "level": "se", "status": "planned" }
  ],
  "segments": [
    { "id": "all", "label": "All products",  "columns": "TOTAL*" },
    { "id": "reg", "label": "Regular",       "columns": "*Reg", "meaning": "product group sold every day" },
    { "id": "fes", "label": "Festive",       "columns": "*Fes", "meaning": "product group sold seasonally" }
  ],
  "period": { "source": "Periode", "granularity": "month" }
}
```

Request and response (placeholders):

```json
{ "principalId": "INA", "period": { "from": "2026-09", "to": "2026-10" },
  "dimensions": ["region", "branch", "spv", "distributor"],
  "measures": ["targetIn", "realIn", "realOut"],
  "filters": { "region": ["TBD-region-key"] },
  "mode": "drill", "drill": { "fromLevel": "region", "path": { "region": "TBD-region-key" } },
  "sort": { "by": "achIn", "dir": "asc" }, "page": { "size": 100, "cursor": null } }
```

```json
{ "rows": [ { "level": 4, "keys": { "region": "TBD", "branch": "TBD", "spv": "TBD", "distributor": "TBD" },
              "values": { "targetIn": 0, "realIn": 0, "realOut": 0 }, "status": { "achIn": "red" } } ],
  "subtotals": [], "grandTotal": { "values": { "targetIn": 0, "realIn": 0, "realOut": 0 }, "scopeNote": "within your access" },
  "dataAsOf": "TBD", "truncated": false }
```

Source columns are Tableau-derived and unverified (`4.2`). All measures are selectable and none is mandatory (DEC-052). Each measure declares the level(s) it exists at: the four sales measures exist from Region to Distributor; `seTarget` and `seSellOut` exist only at the SE level and are planned (DEC-070). A measure that does not exist at a level reads "not available" there. A level whose `measuresFrom` is `null` returns entities only and its measures come back as `"notAvailable"`, never 0. `GET /api/sales-detail/hierarchy` returns the levels, which measures each supports, and the deepest level the signed-in user may open.

### 2.9 Tooltip-ready metric DTOs (`[P]`, supports DASH-001)

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
| `app_config.departments` | `id`, `name`, `sort_order`, `icon`, `status` (`planned`/`active`/`disabled`), `default_landing`, `created_at` | Admin-managed; there is no department table in the business database (DEC-063) |
| `app_config.user_department_access` | `user_id`, `department_id`, `enabled`, `changed_by`, `changed_at` | Enable or disable each department per user (`DEPT-003`) |
| `app_config.module_manifests` | `department_id`, `manifest_json`, `schema_version`, `status` | Validated module manifests (`10-departments-and-modules.md` section 3) |
| `app_config.dashboards` | `id`, `department_id`, `route_key`, `name`, `description`, `visibility`, `published_version`, `created_at` | One row per dashboard; belongs to a department |
| `app_config.dashboard_layout_versions` | `dashboard_id`, `version`, `layout_json`, `schema_version`, `created_by`, `created_at` | Append-only; unique `(dashboard_id, version)` for optimistic concurrency |
| `app_config.achievement_rule_sets` | `version`, `rules_json`, `is_active`, `created_by`, `created_at` | Exactly one active |
| `app_config.playlists` | `id`, `display_id`, `name`, `enabled`, `default_duration_seconds`, `default_presentation`, `on_load_failure` | One playlist per display (for example the CEO TV) |
| `app_config.playlist_items` | `playlist_id`, `dashboard_id`, `position`, `duration_seconds`, `presentation` (`half`/`full`/`auto`), `enabled` | FK to dashboards |
| `app_config.displays` | `id`, `name`, `credential_hash`, `owner_user_id`, `enabled`, `last_seen_at` | A TV or screen with its own revocable credential (DEC-012, DEC-079) |
| `app_config.display_managers` | `display_id`, `user_id` | Who may change that display's playlist and remote-control it |
| `app_config.app_settings` | `key`, `value_json`, `updated_by`, `updated_at` | Platform settings, for example `ui.defaultLanguage` (initial `en`) |
| `app_config.assets` | `id`, `storage_key`, `mime_type`, `size_bytes`, `width`, `height`, `created_by` | Approved storage only |
| `app_config.roles` | `id`, `department_id` (null for platform roles), `name`, `description`, `is_system`, `default_landing`, `created_at` | Admin-managed; seed roles are normal rows; `default_landing` per `MGMT-003` |
| `app_config.role_capabilities` | `role_id`, `capability` | Capability validated against the code catalog |
| `app_config.role_scopes` | `role_id`, `dimension`, `mode` (`all`/`list`), `values_json` | One row per dimension per role |
| `app_config.user_roles` | `user_id`, `role_id` | `user_id` from the identity provider (DEC-007) |
| `app_config.user_scope_overrides` | `user_id`, `dimension`, `add_json`, `remove_json` | Optional |
| `app_config.principals` | `id`, `label`, `tables_json`, `enabled` | Principal registry (2.6) |
| `app_config.users` | `id`, `display_name`, `sign_in_method` (`local` now; `microsoft` later), `enabled`, `created_by`, `created_at` | Created by admins only |
| `app_config.user_mfa` | `user_id`, `enabled`, `method` (`totp`), `secret_encrypted`, `recovery_codes_hash`, `enrolled_at`, `reset_at` | Enabled or disabled per user by an admin |
| `app_config.ai_provider_profiles` | `id`, `name`, `adapter`, `base_url`, `model`, `api_key_encrypted`, `key_last4`, `enabled`, `is_active`, `qualification_status`, `tested_at`, `acknowledged_by`, `created_by` | Key encrypted with a server master key (DEC-049) |
| `app_config.ai_settings` | `enabled`, `temperature`, `max_tool_calls`, `rate_limit`, `retention_days` | Single row |
| `dwh_prod.etl_refresh_log` (**exists**, owned by the ETL; read-only for the app) | `process_name`, `start_time`, `end_time`, `status`, `message` | Source of `dataAsOf` and the refresh signal; the app never writes it (DEC-048) |
| `app_config.saved_views` | `id`, `user_id`, `dashboard_id`, `name`, `view_json`, `is_default` | Pivot layouts and filters (`MGMT-009`) |
| `app_config.usage_events` | `week`, `role_id`, `page`, `action`, `count` | Privacy-safe counts only, no content (`MGMT-018`) |
| `app_config.auth_identities` | `user_id`, `provider`, `tenant_id`, `object_id`, `account_hint`, `linked_at` | **Reserved for deferred Microsoft sign-in** (not used in release 1) |
| `app_config.local_credentials` | `user_id`, `username`, `password_hash`, `must_change`, `failed_attempts`, `locked_until`, `changed_at` | Argon2id hash only |
| `app_config.data_findings`, `data_finding_notes` | See `08-data-findings.md` section 3.1 | Findings and history |
| `app_config.ai_sessions`, `ai_messages`, `ai_results` | `user_id`, question, tool-call summary, result spec JSON, `data_as_of`, timestamps | Per-user chat history and reopenable results; **deleted after 60 days** by a daily job |
| `app_config.audit_events` | `id`, `actor`, `action`, `target`, `summary_json`, `created_at` | Safe summaries only, no secrets; includes permission changes |

Requirements if approved: explicit migrations with constraints, indexes and rollback notes; keep revision history for shared layouts and rules; validate referenced dashboards exist before publishing; never silently overwrite a newer revision.

## 4. Source tables and mapping

### 4.1 Tables supplied by the owner (data dictionary `table_name`)

Source of table names: the repository file `table_name`. Column names come from the Tableau workbook, whose headers are **exactly the PostgreSQL column names** (owner, DEC-001). Data types are confirmed only where stated: target and real values are `int8` (DEC-030); other types and the row grain are checked by the automated schema check (4.6).

| Logical name | INA table | ASW table | Role in the product |
|---|---|---|---|
| Performance (sales, targets) | `dwh_prod.v_tsellinselloutina` | `dwh_prod.v_tsellinselloutasw` | Sell In / Sell Out actual and target per selling point and month |
| Mapping (org + salesman) | `dwh_prod.v_sr_per_branchina` | `dwh_prod.v_sr_per_branch` (confirmed: no suffix means ASW) | Region, branch, area, selling point, salesman (SE) |
| Master distributor | `dwh_prod.vt_sr_per_rsmina` | `dwh_prod.vt_sr_per_rsmasw` | Distributor hierarchy; join target for upload data |
| Date dimension | `dwh_prod.dimdate` | `dwh_prod.dimdate` (shared) | Calendar and working days |
| Active distributors | `dwh_prod.distaktif` (shared today) | same table, filtered by ASW region codes; to be split later (DEC-044) | Total / resigned / active distributors per region and month. **Uploaded manually from Excel; may be stale or contain errors** (see `08-data-findings.md`) |
| Daily upload | `dwh_prod.mt_daily_updateina` | `dwh_prod.mt_daily_updateasw` | Daily upload log per selling point |
| Daily stock upload | `dwh_prod.mt_daily_updatestockina` | `dwh_prod.mt_daily_updatestockasw` | Daily stock upload log |

### 4.2 Columns (names owner-confirmed as exact database names, DEC-001)

- **Meanings confirmed by the owner (DEC-087, DEC-088):** `SLI` = Sell IN (sale **to the distributor**); `SLO` = Sell OUT (sale **from the distributor to the customer**). `RealQtySLI` and `RealQtySLO` are the realised Sell IN and Sell OUT **quantities**; `TargetQtyEskaSLO` is the Sell OUT target by **quantity**; `TargetValueEskaSLO` is the Sell OUT target by **value**; `RealSellInReg/Fes` and the other Reg/Fes columns are **values**; `CUSTNO` is the **customer code** and `CUSTNAME` the **customer name**. There is a quantity target for Sell OUT only; Sell IN has a value target but no quantity target.
- **Product groups:** `Reg` = Regular, products sold every day; `Fes` = Festive, products sold seasonally (owner, DEC-086). `TOTAL...` columns are expected to equal Regular plus Festive; the schema check verifies this.
- **Performance:** `SELLINGPOINTCODE`, `SELLINGPOINTNAME`, `BRANCHCODE`, `BRANCHNAME`, `REGIONCODE`, `REGIONNAME`, `KODEAREA`, `NAMAAREA`, `KOTA`, `Tahun`, `Bulan`, `Periode` (month date), `TargetQtyEskaSLO`, `TargetValueEskaSLO`, `RealQtySLI`, `RealQtySLO`, `RealSellInReg`, `RealSellInFes`, **`TOTALRealSellIn`**, `RealSellOutReg`, `RealSellOutFes`, **`TOTALRealSellOut`**, `TARGETSellInReg`, `TARGETSellInFes`, **`TOTALTargetSellIn`**, `TARGETSellOutReg`, `TARGETSellOutFes`, **`TOTALTargetSellOut`**. Bold = fields used by the four reference sheets.
- **Mapping:** `KOTA`, `REGIONCODE`, `REGIONNAME`, `BRANCHCODE`, `BRANCHNAME`, `KODEAREA`, `NAMAAREA`, `KODESCABANG`, `SELLINGPOINTNAME`, `SLSNO`, `SLSNAME`, `CUSTNO`, `CUSTNAME`, `SLSTP`, `TEAM`, `TEAMNAME`, `BranchActiveFlag`, `FLAG_AKTIF`, `NPWP`, `Source`.
- **Master distributor:** `KODESCABANG`, `BRANCHCODE`, `BRANCHNAME`, `REGIONCODE`, `REGIONNAME`, `KODEAREA`, `NAMAAREA`, `KOTA`, `SELLINGPOINTNAME`.
- **Date dimension:** `date_dim_id`, `dates`, `daysofyears`, `months`, `years`, `quarters`, `monthsname`, `daysofmonths`, `daysofweekname`, `weeknumber`, `workingday(5)`, `workingday(6)`, `daysofweek`, `ganjilgenap`, `holiday`, **`networkeddays`**, `monthyear`.
- **Active distributors:** `RegionCode`, `RegionName`, `AreaCode`, `AreaName`, `Bulan`, `TOTAL DIST`, `Total Resign`, `Total Aktif`.
- **Daily upload / stock upload:** `SELLINGPOINTCODE`, `SELLINGPOINTNAME`, `INVOICE_DATE` (stock: `STOCKUPDATEDATE`), `KOTA`, `Updated`.

Use these names exactly as the **column registry** in code (one file, one place) and always quote identifiers in SQL; never build a column name from user input. If a name differs in the real table (for example case folding, or spaces and parentheses in `TOTAL DIST` and `workingday(5)`), the schema check reports it before any query runs.

### 4.3 Joins observed in the workbook (`[R]`)

- Master distributor `KOTA` = Daily upload `KOTA`.
- Master distributor `KOTA` = Daily stock upload `KOTA`.
- All other tables are used as separate sources blended on region, branch, area, selling point and month in Tableau; the exact join keys are confirmed by the schema check and the owner (DEC-076).

### 4.4 Organisation hierarchy and role mapping (`[R]`, confirm in DEC-003)

| Role | Source level | Evidence |
|---|---|---|
| RSM | `REGIONNAME` (5 regions give 5 RSM) | Region-level achievement flags |
| ASM | `BRANCHNAME` (19) | Branch-level flags |
| SPV | `NAMAAREA` (75) | Area-level flags |
| DIST | `SELLINGPOINTNAME` (confirmed meaning, DEC-002; 144 in the reference) | Selling-point flags |
| SE (sales executive) | Name `SLSNAME`, key `SLSNO` (confirmed, DEC-002). Count: distinct `SLSNO` excluding codes that contain "SOI" in the INA reference sheet (260); a "SOA" variant exists for the other principal. Active flags are not used by Tableau (DEC-028) | `SE Excl SOI` field in sheet `Jumlah Team RSM INA` |
| Customer | Code `CUSTNO`, name `CUSTNAME` (confirmed, DEC-088). The mapping view has no month column, so it is a **current snapshot**, not history | Customer entities under an SE; no customer-level facts |

Hierarchy: Region (RSM) > Branch (ASM) > Area (SPV) > Selling point (DIST) > Salesman (SE) > Customer (entities only, from the mapping). **In the current tables, targets exist only per selling point (distributor).** The owner says the SE-level monthly target and the month-to-date sell-out value per salesperson will be used in the Marketing module, which also enables day-by-day achievement (DEC-059, DEC-070). Whether they already exist in a table, and which one, is open. **Item = product name with product code `pcode`**; it is a separate product dimension that belongs to the Marketing dashboard, added later (DEC-060).

### 4.5 Mapping status per dashboard item

| Item | Source (table, columns) | Grain | Status |
|---|---|---|---|
| Sell IN actual and target | Performance: `TOTALRealSellIn`, `TOTALTargetSellIn` | Selling point per month | Derived from Tableau; DB verification pending |
| Sell Out actual and target | Performance: `TOTALRealSellOut`, `TOTALTargetSellOut` | Selling point per month | Derived from Tableau; DB verification pending |
| Region list and order | Performance / Mapping: `REGIONCODE`, `REGIONNAME` | Region | Display order proposed by the schema check, owner approves (DEC-076) |
| Team totals RSM/ASM/SPV/SE | Mapping: region, branch, area, `SLSNO` | Salesman | Derived; active flags undefined (DEC-028) |
| Role achievement counts | Performance aggregated per entity per month; achieved when achievement >= working-day progress | Entity per month | Rule confirmed (DEC-021); DB verification pending |
| Data refresh time (`dataAsOf`) | `dwh_prod.etl_refresh_log`: `end_time` of the newest `SUCCESS` row for process `Refresh Materialized Views` | Run | Confirmed table and columns (DEC-048); readiness semantics in DEC-077 |
| Sales Detail (pivot) | Performance table via the field catalog (2.8): region, branch, area, selling point, target, real Sell IN, real Sell OUT; period `Periode` | Selling point per month | Derived from Tableau; DB verification pending |
| Working-day progress | `dimdate`: `dates`, `networkeddays`, `monthyear`; summed per month, no custom calendar | Date | Confirmed approach (DEC-027); verify 18 / 27 on 22-Oct-2026 |
| Growth Sell In / Out | Performance, same month last year (12 months earlier) | Region per month | Derived from Tableau |
| Active distributors per region | `distaktif`: `Total Aktif`, `TOTAL DIST`, `Bulan`; % = active / total | Region/area per month | Formula confirmed (DEC-024); DB verification pending |
| Daily upload status | Daily upload: `INVOICE_DATE`, `Updated`, `KOTA`; join Master distributor on `KOTA` | Selling point per day | Derived; stock variant open (DEC-026) |

An item is ready for production use only when its row is `Approved` and has a verified example (definition of ready in `05-roadmap-and-testing.md`). Until then it uses mock fixtures.

### 4.6 Automated schema check (Phase 1 task; replaces the manual queries)

The sandbox that writes this specification cannot reach your database, so the check is a small script in the repository (`pnpm schema:check`), run by the technical administrator with the read-only database user. It reads the column registry and prints one report; nothing is changed in the database.

It checks, for every table in the principal registry:
1. Every registry column exists, with its data type (expected: target and real `int8`) and nullability; reports extra or missing columns and name differences.
2. Row counts and the range of periods (`Periode`) per table and principal.
3. **Grain:** whether the performance view has one row per selling point per month (duplicate-key check), and whether months are complete.
3c. `TargetValueEskaSLO` versus `TOTALTargetSellOut`: equal, or different sources? Quantity columns: types, zeros and negatives; value without quantity or the reverse.
3b. For Sell IN and Sell OUT, target and real: does `TOTAL` equal Regular plus Festive (count of rows that differ, sum of differences)?
4. Distinct regions per principal and a proposed display order (INA expected: JAWA 1, JAWA 2, PULAU 1, SUMATERA 1, SUMATERA 2; ASW six, including SUMATERA 3).
5. `dimdate`: working days for the current month and whether 22-Oct-2026 gives 18 of 27 (66.67%).
6. `dwh_prod.etl_refresh_log`: newest rows per `process_name`, distinct `status` values, whether a running row has a null `end_time`, and whether the timestamps carry a time zone (DEC-077).
7. `distaktif`: months present, regions present per principal, active greater than total, regions missing from the mapping (feeds the findings checks).
8. One hand-checkable figure per principal for the owner to compare with Tableau (a region's Sell IN and Sell Out for one month).

The report is saved as a Markdown file for the owner to review and approve the mapping rows in 4.5.

## 5. Mock fixtures

- Live in `fixtures/`, every file named or headed `MOCK`; never used in production paths and never treated as business evidence.
- Golden test cases (`fixtures/golden/`) cover: 0/0, 0/5, 1/5 = 20%, 5/5, 3 of 5 working days = 60%, the color-rule table in `01-metrics-contract.md`.
- A dev-only synthetic Postgres (docker-compose) may be added for end-to-end testing; it must live in a clearly named dev-only schema and never be deployed.
