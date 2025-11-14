# `scd2_plus`: Advanced SCD Type 2 Materialization for dbt

`scd2_plus` is a custom dbt materialization designed to produce a robust Slowly Changing Dimension Type 2 (SCD2) table. It extends dbt’s snapshot-like behavior with support for deduplication, hybrid Type I/II logic, out-of-order historical changes, and configurable validity windows.

---

## What This Package Provides

- Inserts a new record whenever any column in `check_cols` changes, similar to the `check` snapshot strategy.
- Uses an `updated_at` column (as in the timestamp snapshot strategy) to determine `valid_from` and `valid_to`.
- Handles batch-style historical loads, including duplicate rows and data arriving out of order.
- Deduplicates using `unique_key` and `updated_at`, resolving conflicting values with the latest `loaded_at`.
- Operates incrementally: creates the table if missing, inserts new rows, and updates existing records when required.
- Manages late-arriving events by splitting existing periods to maintain correct temporal boundaries.
- Supports a hybrid SCD model:
  - **`check_cols`** → Type II behavior  
  - **`punch_thru_cols`** → Updated across all versions  
  - **`update_cols`** → Updated in the most recent version only
- Allows full customization of service column names and temporal boundaries.
- Includes a validation test to ensure no gaps or overlaps in `valid_from` / `valid_to`.

---

## Feature Summary

- Type II change tracking with `check_cols`
- Type I updates across all versions via `punch_thru_cols`
- Last-record-only updates using `update_cols`
- Graceful handling of out-of-order records
- Deduplication using `updated_at` and `loaded_at`
- Configurable validity boundaries for each record
- Fully incremental; no full refresh required
- Built-in correctness checks for temporal versioning
- Supports Postgres, Snowflake, BigQuery, Spark, DuckDB, and Trino

---

## Generated Columns

| Purpose                   | Default          | Configurable via                    |
|---------------------------|------------------|-------------------------------------|
| Surrogate key             | `scd_id`         | `scd_id_col_name`                   |
| Validity start timestamp  | `valid_from`     | `scd_valid_from_col_name`           |
| Validity end timestamp    | `valid_to`       | `scd_valid_to_col_name`             |
| Record version number     | `record_version` | `scd_record_version_col_name`       |
| Record load timestamp     | `loaddate`       | `scd_loaddate_col_name`             |
| Record update timestamp   | `updatedate`     | `scd_updatedate_col_name`           |
| Change hash               | `scd_hash`       | Internal only                       |

---

## Installation

Add to `packages.yml`:

```yaml
packages:
  - git: "https://github.com/icygiant/versioned_scd"

Then, run
```bash
dbt deps
```
In your `dbt_project.yml`, add
```yaml
vars:
  loaddate: "1900-01-01"
```
 ## Limitations
❌ No support for soft deletes

❌ No schema auto-evolution

❌ Not compatible with dbt model contracts

## Conceptual Model
* Tracks changes by comparing the current row with existing versions using ```check_cols```.

* Constructs temporal windows using ```updated_at```, managed through ```valid_from``` and ```valid_to```.

* Allows late-arriving data to adjust history by splitting existing periods when necessary.

* Only configured columns are materialized; any non-configured fields in the ```SELECT``` statement are ignored.
