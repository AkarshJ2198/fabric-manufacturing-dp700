# Lab 08 – End-to-End Databricks-to-Fabric Migration Capstone

## Objective
Migrate a representative Databricks workload to Fabric, validate business outcomes and demonstrate failure recovery.

## Scenario
The manufacturing organization is moving sales and inventory processing from Databricks to Fabric. Reporting continuity and data correctness are mandatory.

## Activities
1. Restore the capstone baseline: open the assigned workspace, run the trainer-provided setup, restore source files and Bronze/Silver starter data, and verify counts/schema.
2. Inspect `Assets/ReferenceNotebooks/databricks_customer_migration.py`. Adapt parameters, paths, logging and Delta operations, execute the Fabric version and capture results.
3. Run equivalent input through both references. If no live Databricks workspace is provisioned, use the frozen reference outputs supplied by the trainer.
4. Reconcile counts, business keys, schema, sales/inventory aggregates, current customer rows, SCD2 history and rejected records. Explain intentional differences.
5. Simulate a failed deployment using a controlled incorrect parameter/transformation. Capture failure, inspect audit logs, restore the known-good version from Git or saved baseline and rerun validation.
6. Document cutover entry criteria, parallel-run duration, reconciliation tolerances, business approval, reporting switch, rollback trigger/owner and post-cutover monitoring.
7. Present ingestion, transformation, quality controls, Gold reporting, monitoring and migration validation.

## Deliverables
Working Fabric migration solution, reconciliation report, failure/recovery evidence, cutover/rollback checklist and architecture walkthrough.