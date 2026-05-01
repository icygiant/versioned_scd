# scd2_plus: advanced scd type 2 materialization for dbt

The `scd2_plus` package provides a custom dbt materialization designed for high-fidelity slowly changing dimension type 2 (scd2) implementations. The materialization extends standard dbt snapshot logic by providing native support for deduplication, hybrid persistence models, and non-sequential historical ingestion.

---

## functional capabilities

* **deterministic change detection**: generates a new record version when any column defined in `check_cols` changes, mirroring the dbt `check` strategy.
* **temporal interval management**: uses an `updated_at` source column to define the `valid_from` and `valid_to` boundaries for each record.
* **historical ingestion support**: handles batch-style historical loads, including duplicate records and data arriving out of chronological order.
* **conflict resolution**: implements deduplication using a composite of `unique_key` and `updated_at`, resolving collisions via the latest `loaddate`.
* **incremental processing**: supports standard incremental patterns, including table initialization, record insertion for new states, and updates to existing rows to maintain temporal integrity.
* **late-arriving event handling**: maintains temporal continuity for late-arriving data by automatically splitting existing intervals to accommodate new historical states.
* **multi-state persistence model**:
    * **type II (check_cols)**: provides full versioning for historical tracking.
    * **type I (punch_thru_cols)**: applies global updates across all historical versions of a record.
    * **current-state (update_cols)**: applies updates exclusively to the most recent version of a record.
* **integrity validation**: includes automated testing to identify gaps or overlaps within the `valid_from` and `valid_to` sequences.

---

## technical specifications


| purpose | default column name | configuration variable |
| :--- | :--- | :--- |
| primary surrogate key | `scd_id` | `scd_id_col_name` |
| interval start timestamp | `valid_from` | `scd_valid_from_col_name` |
| interval end timestamp | `valid_to` | `scd_valid_to_col_name` |
| version index | `record_version` | `scd_record_version_col_name` |
| ingestion timestamp | `loaddate` | `scd_loaddate_col_name` |
| update timestamp | `updatedate` | `scd_updatedate_col_name` |
| change vector hash | `scd_hash` | internal only |


### supported adapters

The materialization is compatible with postgres, snowflake, bigquery, spark, duckdb, and trino.

---

## installation

Add the package to your `packages.yml` file:

```yaml
packages:
  - git: "https://github.com/icygiant/versioned_scd"
```

Update your project dependencies:
```bash
dbt deps
```

Define the baseline load date in your `dbt_project.yml`:
```yaml
vars:
  loaddate: "1900-01-01"
```

---

## limitations

* **soft deletes**: the materialization does not support the detection or management of records flagged as 'deleted' but not actually erased from the source.
* **schema evolution**: automated schema modifications, such as adding or removing columns, are not supported.
* **model contracts**: the implementation is currently incompatible with dbt model contracts.
* **field scope**: only columns explicitly defined in the configuration are materialized; any extra fields in the `select` statement are ignored.