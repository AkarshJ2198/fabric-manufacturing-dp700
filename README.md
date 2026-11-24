# Microsoft Fabric DP-700 – Manufacturing & Supply Chain Labs

A hands-on, eight-day (32-hour) training repository focused on building and operating a Microsoft Fabric data engineering solution for a fictional manufacturing and supply-chain business, including a Databricks-to-Fabric modernization scenario.

> **Training data only:** all supplied records are synthetic. Do not add client production data, credentials, access tokens, connection strings, or other secrets to this public repository.

## Start here
1. Review [Environment Setup](Instructions/Labs/00-Environment-Setup.md).
2. Confirm your Skillable tenant, workspace, capacity, Lakehouse, Warehouse, Spark, Data Factory, Git integration, and permissions with the instructor.
3. Complete the labs in numerical order. Each lab guide identifies the assets and expected evidence for that session.
4. At the end of every session, export your work to a persistent location approved by your instructor. Do not rely on the Skillable VM or Fabric workspace persisting after the lab closes.

## Eight-day learning path
| Day | Lab | Focus |
|---|---|---|
| 1 | [Architecture and migration assessment](Instructions/Labs/01-Architecture-and-Migration-Assessment.md) | Manufacturing data domains, Databricks-to-Fabric mapping, migration assessment |
| 2 | [Batch ingestion and Bronze](Instructions/Labs/02-Batch-Ingestion-and-Bronze.md) | Source files, Lakehouse, ingestion, Bronze layer |
| 3 | [Metadata-driven incremental load](Instructions/Labs/03-Metadata-Driven-Incremental-Load.md) | Configuration-driven orchestration, watermarks, incremental loads |
| 4 | [Delta MERGE, SCD and schema evolution](Instructions/Labs/04-Delta-Merge-SCD-and-Schema-Evolution.md) | Upserts, deduplication, SCD patterns, schema drift |
| 5 | [Gold, Warehouse and Power BI](Instructions/Labs/05-Gold-Warehouse-and-PowerBI.md) | Curated models, SQL analytics, semantic/reporting continuity |
| 6 | [Data quality, reconciliation and performance](Instructions/Labs/06-Data-Quality-Quarantine-Reconciliation-Performance.md) | Validation, quarantine, audit, Spark and capacity optimization |
| 7 | [Security, Git and deployment](Instructions/Labs/07-Security-Git-CICD-and-Deployment.md) | Workspace access, governance, source control and release practices |
| 8 | [End-to-end migration capstone](Instructions/Labs/08-End-to-End-Migration-Capstone.md) | Integrated pipeline, validation, parallel run, cutover and rollback |

## Repository layout
- `Instructions/Labs/` – participant-facing lab guides.
- `Assets/Datasets/` – synthetic manufacturing, customer, product, supplier, inventory, production and sales data, including incremental and exception files.
- `Assets/Metadata/` – sample ETL configuration and watermark files.
- `Assets/ReferenceNotebooks/` – representative Databricks source notebook for migration discussion.

## Skillable persistence and backup
Skillable environments can be temporary. Confirm separately whether the VM disk, Fabric workspace, Lakehouse tables/files, notebooks, pipelines, Warehouse objects and OneLake data persist after an exercise ends. These are distinct persistence boundaries.

At the end of every lab, preserve notebook source, SQL scripts, pipeline definitions, configuration, validation outputs, run IDs and required screenshots in the client-approved LMS, SharePoint, OneDrive or other persistent location. GitHub is the permanent source for starter materials; it is not a substitute for backing up Lakehouse data or every Fabric artifact.

## Important scope note
This repository is a training starter pack. Validate all steps against the specific Skillable image, Fabric tenant configuration, capacity, permissions and current Fabric experience before class delivery. Some exercises require instructor-led environment setup and may need adjustment for the lab tenant.