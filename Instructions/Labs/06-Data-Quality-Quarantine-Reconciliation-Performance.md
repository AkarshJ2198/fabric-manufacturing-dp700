# Lab 06 – Data Quality, Quarantine, Reconciliation and Performance

## Objective
Implement reusable data quality controls and operational monitoring.

## Files required
`Assets/Datasets/Customer_Exceptions.csv` and `Assets/Datasets/Sales_Exceptions.csv`.

## Activities
1. Upload exception files to Bronze `Files/Exceptions`.
2. Implement rules: CustomerID not null, CustomerName not null, unique customer key, SalesOrderID not null, ProductID exists, Quantity > 0 and OrderDate not in the future.
3. Split valid and invalid rows. Record source ID, batch ID, rule, pass/fail, rejection reason and validation timestamp. Write invalid rows to `dq_quarantine`.
4. Reconcile source, Bronze, valid Silver and quarantined counts. For complete classification, verify `SourceCount = ValidCount + QuarantinedCount`.
5. Compare sales amount and quantity aggregates.
6. Record pipeline/run ID, table, batch, status, source/inserted/updated/rejected counts, timestamps and error details.
7. Inspect Spark explain plans and metrics for filter pushdown, join strategy, partition pruning, skew, small files and shuffles. Discuss compaction and capacity utilization; run optimization commands only when supported.

## Deliverables
Validation notebook, `dq_validation_results`, `dq_quarantine`, `etl_reconciliation`, `etl_audit_log` and performance observations.

## Self-paced task
Add a supplier-status rule and show that the framework records it without changing the overall processing pattern.