# Lab 03 – Metadata-Driven Incremental Ingestion

## Objective
Build a reusable ingestion pattern driven by configuration and watermark metadata.

## Files required
- `Assets/Metadata/etl_config.csv`
- `Assets/Metadata/etl_watermark.csv`
- `Assets/Datasets/Customer_Incremental.csv`
- `Assets/Datasets/SalesOrder_Incremental.json`

## Activities
1. Upload metadata files to `LH_MFG_BRONZE/Files/Metadata`.
2. Read them and create Delta tables `etl_config` and `etl_watermark`.
3. Upload incremental files to `Files/Incremental`.
4. Inspect Customer changes and apply a `ModifiedDate` watermark filter.
5. Deduplicate each incoming batch by business key and latest modification timestamp.
6. MERGE into the designated current-state target; verify reruns are idempotent.
7. Record batch ID, start/end time, row count and status in an audit table.
8. Update the watermark only after successful processing.
9. Build a Data Factory pipeline with Lookup, ForEach, Notebook activity, success/failure paths, audit and retry handling. Parameterize table, source, watermark column and batch ID.

Example metadata load:

```python
config = (spark.read.option("header", True).option("inferSchema", True)
          .csv("Files/Metadata/etl_config.csv"))
watermarks = (spark.read.option("header", True).option("inferSchema", True)
              .csv("Files/Metadata/etl_watermark.csv"))
config.write.format("delta").mode("overwrite").saveAsTable("etl_config")
watermarks.write.format("delta").mode("overwrite").saveAsTable("etl_watermark")
```

## Deliverables
`etl_config`, `etl_watermark`, `etl_audit_log`, incremental notebook, pipeline screenshot and run result.

## Self-paced task
Add a metadata row for a new source and process it without changing the orchestration design.