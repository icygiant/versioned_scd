# `scd2_plus`: Advanced SCD Type 2 Materialization for dbt

`scd2_plus` is a custom dbt materialization that builds a robust Slowly Changing Dimension Type 2 (SCD2) table. It enhances dbt's snapshot-like behavior with support for deduplication, hybrid Type I/II fields, out-of-order historical changes, and configurable validity handling.

---

## 📦 What does this dbt package do?

- A new record is added if there is a change in the `check_cols` column list — just like in `check` strategy of dbt snapshots.
- It uses an `updated_at` column like the timestamp snapshot strategy to define the `valid_from` and `valid_to` columns.
- Supports batch-style historical initial loads, including duplicates and reordering based on `updated_at`.
- Deduplicates rows using `unique_key` + `updated_at`. If conflicting values exist, resolves using the latest `loaded_at`.
- Incrementally loads data: creates table if it doesn't exist, inserts new rows, and updates existing ones when necessary.
- Handles **out-of-order inserts**: if a late-arriving record overlaps with already loaded data, it splits the relevant period.
- Supports hybrid SCD Type I and Type II logic:
  - `check_cols` → Type II columns
  - `punch_thru_cols` → updated across **all versions**
  - `update_cols` → updated in **last version only**
- Supports fully customizable service column names and temporal windows.
- Validation test ensures there are no gaps or overlaps in the `valid_from` / `valid_to` periods.

---

## 🚀 Features

- ✅ Type II change tracking with `check_cols`
- 🔁 Type I updates across all versions with `punch_thru_cols`
- 🧩 Last-record-only updates with `update_cols`
- ⏪ Handles out-of-order records gracefully
- 📥 Deduplicates using `updated_at` + `loaded_at`
- 📅 Configurable open/close bounds for surrogate record time ranges
- ⚙️ Works incrementally — no full refresh needed
- 🧪 Built-in validation test for versioning correctness
- 🧬 Supports Postgres, Snowflake, BigQuery, Spark, DuckDB, Trino

---

## 🧠 Generated Columns

| Purpose                    | Default                | Configurable with                |
|---------------------------|------------------------|----------------------------------|
| Surrogate key             | `scd_id`               | `scd_id_col_name`                |
| Validity start timestamp  | `valid_from`           | `scd_valid_from_col_name`        |
| Validity end timestamp    | `valid_to`             | `scd_valid_to_col_name`          |
| Record version number     | `record_version`       | `scd_record_version_col_name`    |
| Record load timestamp     | `loaddate`             | `scd_loaddate_col_name`          |
| Record update timestamp   | `updatedate`           | `scd_updatedate_col_name`        |
| Change hash               | `scd_hash`             | Internal only                    |

---

## 📦 Installation

In your `packages.yml`:

```yaml
packages:
  - git: "https://github.com/icygiant/versioned_scd"
```
Then, run
```bash
dbt deps
```
In your `dbt_project.yml`, add
```yaml
vars:
  loaddate: "1900-01-01"
```
 ## 🛑 Limitations
❌ No support for soft deletes

❌ No schema auto-evolution

❌ Not compatible with dbt model contracts

## Conceptual Model
- Tracks entity changes by comparing current values with prior versions via check_cols.

- Period windows are generated using updated_at and managed via valid_from and valid_to.

- Late-arriving data can modify historical windows — records may be split.

- Only configured columns are materialized — all others in the SELECT are ignored.
