# Data Findings (Anomalies to Validate)

Related: [`PRD.md`](../PRD.md) · [`01-metrics-contract.md`](01-metrics-contract.md) · [`06-dashboard-catalog.md`](06-dashboard-catalog.md)

Tags: `[C]` owner-confirmed, `[P]` proposed, `[O]` open.

## 1. Why this exists

The owner explained (DEC-034) that two numbers on the dashboards come from different sources:

- **DIST total (for example 144):** counted from the database (distinct selling points/branches in the mapping data).
- **Active distributors (Total Aktif / TOTAL DIST):** from table `distaktif`, which is **uploaded manually from Excel**. It may be out of date or contain human error.

The two will not always agree (144 against 120 on the reference screenshots). That is a **finding to validate**, not an error to hide or silently fix. The owner asked for a separate sheet that lists anomalies and findings that need validation.

## 2. Requirements

- `FIND-001` `[C]` A separate dashboard/sheet **"Data Findings"** lists anomalies and items needing validation, per principal, region and month.
- `FIND-002` `[C]` The app **never changes or hides source numbers** because of a finding. It shows the data as stored, with a visible note when a finding applies.
- `FIND-003` `[C]` Source provenance is shown where it matters: for example the active-distributor figure is labelled "manual upload (`distaktif`)" with the latest month present in the table.
- `FIND-004` `[P]` Dashboards show a small data-quality indicator on affected elements; clicking it opens the related finding.
- `FIND-005` `[P]` The AI chat includes relevant open findings as warnings in its answers (`AI-016`).
- `FIND-006` `[C]` Visibility and management are permissions: `findings.view` and `findings.manage`, assignable per role by admins, and limited by the user's data scope.

## 3. Initial checks (all `[P]`; thresholds `[O]`, DEC-043)

| Code | Check | Example finding |
|---|---|---|
| `DIST_COUNT_MISMATCH` | Distinct selling points per region in the mapping data vs `TOTAL DIST` in `distaktif` | Mapping 144 vs `distaktif` 120; per-region differences listed |
| `DISTAKTIF_STALE` | Latest `Bulan` in `distaktif` is older than the selected or current month | "No upload for Oct-26" |
| `DISTAKTIF_INCONSISTENT` | `Total Aktif` greater than `TOTAL DIST`; `Total Aktif + Total Resign` different from `TOTAL DIST`; negative values; region in `distaktif` absent from mapping | Active 35 of 31 |
| `TARGET_MISSING_OR_ZERO` | Actual greater than 0 with missing or zero target; negative actual or target | Region with no target for the month |
| `ACHIEVEMENT_OUTLIER` | Achievement above an agreed ceiling (threshold open) | 644.95% growth values in a region |
| `MAPPING_MISMATCH` | Region, branch or selling point present in performance data but absent from mapping, or the reverse; the same salesman number mapped to several regions | Unknown branch name |
| `UPLOAD_GAP` | Distributor with no upload for N consecutive days (threshold open) | Daily Upload dashboard |
| `SELL_OUT_ABOVE_SELL_IN` | Sell OUT far above Sell IN for a distributor over several months (threshold DEC-043) | Possible mapping or upload error |
| `QTY_VALUE_MISMATCH` | Quantity with no value, or the reverse (when quantity columns are registered, DEC-087) | Real value 0 with quantity above 0 |
| `SCHEMA_DRIFT` | A registered column is missing, renamed or retyped after an ETL run (`OPS-021`) | Column dropped by the ETL |
| `PARTIAL_DATA` | A fresh result is implausibly small compared with the last good one (`OPS-020`) | Current month rows missing |
| `USER_REPORTED` | A user flags a number from any chart ("report anomaly") with a note | Free text |

Checks run after each completed data refresh (`OPS-009`) and on demand; results are stored with the version of the check. A check that cannot run (missing table or column) produces a finding of its own instead of failing silently.

## 3.1 Finding record (`[P]`)

```json
{
  "id": "find-0001",
  "checkCode": "DIST_COUNT_MISMATCH",
  "principalId": "INA",
  "regionId": "TBD-region-key",
  "month": "2026-10",
  "severity": "warning",
  "expected": 0,
  "found": 0,
  "difference": 0,
  "sources": ["dwh_prod.v_sr_per_branchina", "dwh_prod.distaktif"],
  "detectedAt": "TBD",
  "status": "open",
  "assignee": null,
  "notes": [],
  "validatedBy": null,
  "validatedAt": null
}
```

Numbers above are placeholders. Status flow: `open` -> `validated` (confirmed real issue) / `false_positive` / `fixed` (source corrected) -> reopened automatically if the check fails again for the same item. Severity: `info`, `warning`, `critical` (rules open, DEC-043).

## 4. Sheet layout (`[P]`)

- Filters: principal, region, month, severity, status, check code (respecting user scope).
- Table: finding, scope, expected vs found vs difference, source tables, detected at, status, assignee; row expands to history and notes.
- Summary cards: open by severity, newest, oldest open.
- Actions (`findings.manage`): set status, assign, add note, export `[O]`.
- Each finding links to the dashboard element and period it affects.

## 5. Tests

- Seed fixtures that trigger each check (mismatch 144 vs 120, stale month, active greater than total, missing target).
- A finding never alters the dashboard numbers (assert numbers unchanged).
- Scope tests: a user sees only findings for allowed principals and regions.
- Status transitions and automatic reopening.
