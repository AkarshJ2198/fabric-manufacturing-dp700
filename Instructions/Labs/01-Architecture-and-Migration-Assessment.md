# Lab 01 – Architecture Assessment and Migration Planning

## Objective
Assess a representative Databricks Medallion solution and design its Microsoft Fabric target architecture.

## Scenario
A manufacturing company consolidates customer, product, supplier, sales, inventory and production data. The current platform uses Databricks notebooks, Delta tables and scheduled jobs. The target platform is Microsoft Fabric.

## Activities
1. Review the customer, product, supplier, plant, sales order, sales detail, inventory and production source assets.
2. Inspect the reference Databricks notebook and identify platform-specific features such as `dbutils.widgets` and `dbutils.fs`.
3. Create Bronze, Silver and Gold Lakehouses; create a Warehouse if enabled.
4. Create a migration mapping with source component, current purpose, Fabric target, migration action, validation approach and risk/dependency.
5. Map DBFS/cloud storage, Delta tables, PySpark notebooks, Databricks Workflows, Databricks SQL, Unity Catalog concepts and Power BI semantic models.
6. Draw a target architecture containing ERP/CSV/JSON sources, Fabric Data Factory, Lakehouses, Warehouse, Power BI, monitoring, security and Git/CI-CD.
7. Record feature restrictions in the Skillable environment.

## Deliverables
- Target architecture diagram
- Databricks-to-Fabric mapping table
- Workspace and capacity readiness notes

## Self-paced task
Identify five migration risks and propose a mitigation for each.