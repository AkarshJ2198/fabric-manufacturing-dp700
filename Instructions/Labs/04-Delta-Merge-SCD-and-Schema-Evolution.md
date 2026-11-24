# Lab 04 – Delta MERGE, Deduplication, SCD and Schema Evolution

## Objective
Convert a representative Databricks PySpark transformation into a Fabric notebook and implement Delta processing patterns.

## Activities
1. Review the reference Databricks notebook. Identify `dbutils.widgets`, `dbutils.fs`, Spark operations, Delta MERGE, logging and error handling.
2. Replace platform-specific parameters with Fabric notebook/pipeline parameters for source, target and batch ID.
3. Deduplicate by business key using `row_number()` ordered by `ModifiedDate` descending.
4. Use `delta.tables.DeltaTable` to MERGE matched updates and unmatched inserts. Test a repeated batch for idempotency.
5. Implement SCD Type 1 for Product: update changed attributes and insert new products.
6. Implement SCD Type 2 for Customer with `StartDate`, `EndDate`, `IsCurrent` and `RecordHash`. Expire the prior current row and insert a new row for tracked changes. Verify exactly one current record per CustomerID.
7. Read `Product_Incremental.json`, which introduces `ProductSubCategory`. Compare schemas and decide whether to reject, apply a controlled schema change or use supported Delta schema evolution.

## Deliverables
Fabric-adapted notebook, deduplicated target, SCD1 Product, SCD2 Customer and schema evolution validation.

## Self-paced task
Implement SCD Type 2 for Supplier and prove historical rows remain queryable.