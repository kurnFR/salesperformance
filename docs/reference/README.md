# Reference Materials

| File | What it is |
|---|---|
| `sheet1.png` | Sheet 1: Sales Performance dashboard (INA) |
| `sheet2-growth sell in.png` | Sheet 2: Growth Sell In (this year vs last year, rolling 7 months) |
| `sheet3-growth sell out.png` | Sheet 3: Growth Sell Out (same layout as sheet 2) |
| `sheet 4-Daily Upload.png` | Sheet 4: Log Upload SLO Distributor to Eskalink |
| `Analisa Kinerja Sales mockup.twbx` | The Tableau workbook (live CSV/table connections; images only, no data extract) |
| `tableau-formula-extract.md` | Curated extract of the workbook's calculated fields, with notes (generated) |
| `../../table_name` | Data dictionary: CSV name to PostgreSQL table, per principal (INA, ASW) |

The screenshots show the arrangement, colors and labels the app must reproduce. They are not evidence of schema. Column names come from the Tableau CSV connections and must be verified against `information_schema.columns` in PostgreSQL (see `docs/03-data-model.md` section 4).

Do not commit credentials or real customer/sales data. The workbook currently contains only images and connection metadata.
