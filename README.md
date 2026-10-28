# Microsoft Fabric DP-700 labs

Hands-on labs for the **DP-700: Microsoft Fabric Data Engineer Associate** exam, built in my own Fabric workspace and version-controlled with **Fabric Git integration**.

Everything in the [`fabric`](fabric) folder is committed straight from the workspace by Fabric: notebooks are stored as code (`notebook-content.py`) and Lakehouses as definitions. The data itself stays in OneLake.

## How it's set up

- Fabric trial capacity (UK South), workspace **DP700-Labs**
- GitHub sync enabled through the Fabric admin portal tenant settings, using a dedicated Fabric Administrator account (least privilege)
- Workspace connected to this repo with a fine-grained token scoped to this repository only (Contents: read and write)
- After each lab: **Source control → Commit** from the workspace

## Lab log

| # | Lab | What I built | Status |
|---|-----|--------------|--------|
| 01 | Analyse data with Apache Spark | Loaded three years of sales CSVs with an explicit schema, added Year/Month columns, saved partitioned Parquet, queried with Spark SQL | ✅ Done |
| 02 | Ingest data with a pipeline | | ⏳ Next |

## Problems solved

- ### Lab 01

- **CSV with no header row:** `header=true` turned the first order into column names and broke `display()`. Fixed with `header=false` and a defined schema.
- **NameError in the lab script:** the partitioning step referenced `orders_df`, which was never created. Fixed by using `transformed_df`.
- **GitHub option greyed out in Fabric:** GitHub sync is off by default at tenant level. Assigned the Fabric Administrator role and enabled the Git integration tenant settings.
  
- ### Lab 02
  
– Alternating NULL rows after pipeline ingestion

**Symptom:** The `sales` table showed every second row as NULL, with a lone `"` in `SalesOrderNumber`. The copied `Files/new_data/sales.csv` had a `"` on its own line after every record.

**Cause:** The lab instructs setting the Copy data source **Row delimiter** to **Line feed (\n)**, but the source CSV uses Windows line endings (`\r\n`). Splitting on `\n` only left a stray `\r` at the end of each row's last column. On write, the value was quoted with the `\r` inside it, so Spark read each record as two rows.

**Fix:** Copy data → Source → File format settings → **Row delimiter: Default (\r, \n, or \r\n)**. Deleted the bad tables and re-ran the pipeline. Verified with **Preview data** before loading.

**Lesson:** Always preview ingested files before trusting the table. Row/column delimiter, quote character and header settings in the Copy activity directly affect data quality downstream.

## Skills

PySpark · Delta Lake · Fabric Lakehouse · Spark SQL · partitioning · Fabric Git integration · tenant settings · least-privilege access
