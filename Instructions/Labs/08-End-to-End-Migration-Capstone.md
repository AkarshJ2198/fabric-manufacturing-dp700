# Lab 08 – End-to-End Manufacturing Migration Capstone

**Time:** 4 hours  
**Level:** Intermediate  
**Goal:** Demonstrate a complete Fabric data engineering flow and explain how you would migrate, validate, cut over and roll back a Databricks workload.

## 1. Capstone scenario

The manufacturer is moving its sales and inventory processing from Databricks to Fabric. The business requires:
- Complete and correct data
- Incremental processing
- Customer history
- Data quality controls
- Reporting continuity
- Operational monitoring
- A safe rollback plan

## 2. Confirm the starting point

Before beginning, confirm the following exist:
- Bronze source files and tables from Lab 02
- Metadata and incremental processing outputs from Lab 03
- Product Type 1 and Customer Type 2 outputs from Lab 04
- Gold dimensions/facts and report or SQL validation from Lab 05
- Quality, quarantine and reconciliation outputs from Lab 06

If the Skillable environment has been reset, follow Lab 00 and the trainer's baseline restore instructions. Do not assume Fabric workspace data survives when the Skillable VM closes.

## 3. Draw the end-to-end architecture

Draw a diagram with these components:
1. ERP/source files
2. OneLake landing area
3. Bronze Lakehouse
4. Silver Lakehouse
5. Gold Lakehouse or Warehouse
6. Fabric Data Factory orchestration
7. Notebooks for transformation
8. Audit and data quality tables
9. Power BI semantic model/report
10. Git/source control and monitoring

Label the data flow and identify where failures are logged.

## 4. Run the end-to-end process

1. Open the ingestion notebook and run the Bronze load cells.
2. Confirm all required Bronze tables exist.
3. Run the incremental Customer notebook cells.
4. Confirm the Customer current-state table has the latest version.
5. Run the SCD2 processing cells and verify one current row per CustomerID.
6. Run the Gold model notebook.
7. Run the data quality notebook.
8. Confirm the reconciliation table shows PASS.
9. Open the Power BI report or use the Gold SQL queries if report authoring is unavailable.
10. Record each notebook run status, row counts and execution time.

Run cells in order. If a cell fails, capture the full error before changing code.

## 5. Compare Databricks and Fabric

1. Open `Assets/ReferenceNotebooks/databricks_customer_migration.py`.
2. Identify the source logic, parameters, paths, transformation and output table.
3. Open the corresponding Fabric notebook.
4. Create a mapping table with: Databricks operation, Fabric implementation, differences, validation evidence.
5. Compare source and target:
   - Schema and data types
   - Row counts
   - Distinct business keys
   - Null counts for key columns
   - Sales amount and quantity totals
   - Current Customer records and historical versions
   - Rejected record counts

If a live Databricks workspace is not provided, compare against the documented source behavior and clearly mark the comparison as a design review rather than a live parallel run.

## 6. Simulate a controlled failure

1. Create a copy of your notebook named `NB_08_FAILURE_TEST`.
2. In the copy only, change a table name to a deliberately incorrect name, such as `brz_customer_typo`.
3. Run the affected cell.
4. Confirm that the run fails with a table-not-found error.
5. Capture the error and note the failed activity/run ID if available.
6. Restore the correct table name.
7. Rerun the cell.
8. Confirm the notebook succeeds.
9. Do not introduce the deliberate error into the shared baseline notebook.

## 7. Define cutover entry criteria

Complete the following table:

| Criterion | Example evidence | Pass/Fail |
|---|---|---|
| Data completeness | Source and target counts reconcile | |
| Data correctness | Key and aggregate checks pass | |
| Incremental correctness | Repeated batch does not duplicate data | |
| Historical correctness | SCD2 current/history checks pass | |
| Data quality | Quarantine and reconciliation reviewed | |
| Performance | Run time meets agreed target | |
| Security | Access and permissions approved | |
| Reporting | Business totals and visuals validated | |
| Operations | Monitoring and support owner confirmed | |

## 8. Define parallel run and rollback

Write a short plan covering:
- How long Databricks and Fabric will run in parallel
- Which tables and business aggregates will be compared
- Who signs off on differences
- What tolerance is allowed for timing-related differences
- Who approves the cutover
- What event triggers rollback
- How reports and source schedules are switched back
- How data written during the failed cutover is reconciled

Do not assume a rollback is simply switching a report connection. Consider data writes, pipeline schedules, watermarks, permissions and downstream dependencies.

## 9. Final presentation

Present for 5–10 minutes:
1. Source and target architecture
2. Bronze ingestion
3. Incremental processing and MERGE
4. SCD1/SCD2 behavior
5. Data quality and reconciliation
6. Gold model and reporting
7. Monitoring and failure recovery
8. Cutover and rollback recommendation

## 10. Final submission checklist

- [ ] Architecture diagram
- [ ] Successful end-to-end run evidence
- [ ] Source-to-target reconciliation
- [ ] Incremental rerun/idempotency evidence
- [ ] SCD2 current/history validation
- [ ] Data quality and quarantine results
- [ ] Failure simulation and recovery evidence
- [ ] Cutover and rollback plan
- [ ] Final presentation or walkthrough completed