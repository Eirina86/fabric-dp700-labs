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
| 02 | Ingest data with a pipeline | Copy data, Delete data, Notebook activity, base parameters | ✅ Done |
| 03 | Use Delta tables in Apache Spark | Managed & external tables, table versioning/time travel, Spark SQL queries, Structured Streaming into a Delta table | ✅ Done |
| 04 | Create and use Dataflows (Gen2) | Power Query transformations, custom column, lakehouse destination (Append), dataflow activity in a pipeline | ✅ Done |

## Problems solved

---

### 🔹 Lab 01 – Analyse data with Apache Spark

**CSV with no header row:** `header=true` turned the first order into column names and broke `display()`. Fixed with `header=false` and a defined schema.

**NameError in the lab script:** the partitioning step referenced `orders_df`, which was never created. Fixed by using `transformed_df`.

**GitHub option greyed out in Fabric:** GitHub sync is off by default at tenant level. Assigned the Fabric Administrator role and enabled the Git integration tenant settings.

---

### 🔹 Lab 02 – Ingest data with a pipeline

#### Alternating NULL rows after pipeline ingestion

**Symptom:** The `sales` table showed every second row as NULL, with a lone `"` in `SalesOrderNumber`. The copied `Files/new_data/sales.csv` had a `"` on its own line after every record.

**Cause:** The lab instructs setting the Copy data source **Row delimiter** to **Line feed (\n)**, but the source CSV uses Windows line endings (`\r\n`). Splitting on `\n` only left a stray `\r` at the end of each row's last column. On write, the value was quoted with the `\r` inside it, so Spark read each record as two rows.

**Fix:** Copy data → Source → File format settings → **Row delimiter: Default (\r, \n, or \r\n)**. Deleted the bad tables and re-ran the pipeline. Verified with **Preview data** before loading.

**Lesson:** Always preview ingested files before trusting the table. Row/column delimiter, quote character and header settings in the Copy activity directly affect data quality downstream.

#### Duplicate rows after re-running the notebook

**Symptom:** `sales` had 65,436 rows; `new_sales` had 32,718.

**Cause:** The notebook writes with `.mode("append")`, so each run added the full dataset again.

**Fix:** `DELETE FROM dbo.sales` in Spark SQL, re-ran once, verified counts via the SQL analytics endpoint.

**Lesson:** Use `overwrite` or `MERGE` for re-runnable loads.

#### Spark capacity error 430 (TooManyRequestsForCapacity)

**Cause:** A notebook and a Spark SQL query each held a Spark session on the trial capacity.

**Fix:** Stopped idle sessions / cancelled in Monitor hub; used the SQL analytics endpoint (no Spark) for read-only checks.

---

### 🔹 Lab 03 – Use Delta tables in Apache Spark

#### TABLE_OR_VIEW_NOT_FOUND

**Cause:** Lab code uses example names (`dp_workspace.delta_lakehouse.dbo.products_table`) that don't match my workspace and lakehouse.

**Fix:** Used just the table name, since the notebook's default lakehouse is attached.

**Lesson:** Lab text and screenshots use different example names; always substitute your own.

---

### 🔹 Lab 04 – Create and use Dataflows (Gen2)

#### Default destination couldn't be edited

**Observation:** Creating the dataflow from the lakehouse added a default destination (Replace, auto mapping). It can only be viewed, not edited, and it greys out *Add data destination* because a query can only have one destination.

**Fix:** Removed the default destination (✕) and re-added Lab04_Lakehouse with **Append**, as the lab instructs.

#### Duplicate rows from Append

**Symptom:** `orders` had 1,084 rows but only 542 unique (SalesOrderID + LineItem).

**Cause:** The dataflow ran twice (Save & run, then the pipeline), and Append added the full dataset each time.

**Check:** `COUNT(*)` vs `COUNT(DISTINCT CONCAT(SalesOrderID, '-', LineItem))` on the SQL analytics endpoint.

**Lesson:** Append keeps adding; Replace overwrites. Same trade-off as Spark `append` vs `overwrite`.

---



## Skills

PySpark · Delta Lake · time travel · Structured Streaming · Fabric Lakehouse · Spark SQL · partitioning · Data pipelines · SQL analytics endpoint · Fabric Git integration · tenant settings · least-privilege access
