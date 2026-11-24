# Microsoft Fabric DP-700 – Manufacturing & Supply Chain Labs

A hands-on, eight-day (32-hour) training repository focused on building and operating a Microsoft Fabric data engineering solution for a fictional manufacturing and supply-chain business, including a Databricks-to-Fabric modernization scenario.

> **Training data only:** use synthetic data. Never commit client production data, credentials, access tokens, connection strings, personal information or other secrets to this public repository.

## Start here

1. Complete [Lab 00 – Environment Setup](Instructions/Labs/00-Environment-Setup.md).
2. Complete Labs 01–08 in order. The guides are written as beginner walkthroughs with prerequisites, navigation steps, code cells, validation checks, troubleshooting and homework.
3. Keep your notebook and run evidence in a persistent location approved by the instructor. Skillable VMs and Fabric workspaces may have different reset/persistence behavior.
4. Do not skip a lab's completion checklist. Ask the instructor when your Skillable tenant does not expose a feature mentioned in a guide.

## Eight-day learning path

| Day | Lab | Focus |
|---|---|---|
| 1 | [Architecture and migration assessment](Instructions/Labs/01-Architecture-and-Migration-Assessment.md) | Source data, relationships, Databricks-to-Fabric mapping and target architecture |
| 2 | [Batch ingestion and Bronze](Instructions/Labs/02-Batch-Ingestion-and-Bronze.md) | Upload files, read with PySpark, add audit metadata and create Delta tables |
| 3 | [Metadata-driven incremental load](Instructions/Labs/03-Metadata-Driven-Incremental-Load.md) | Configuration, watermarks, deduplication, MERGE, audit and orchestration |
| 4 | [Delta MERGE, SCD and schema evolution](Instructions/Labs/04-Delta-Merge-SCD-and-Schema-Evolution.md) | Type 1, Type 2, schema drift and validation |
| 5 | [Gold, Warehouse and Power BI](Instructions/Labs/05-Gold-Warehouse-and-PowerBI.md) | Dimensions, facts, aggregates and reporting |
| 6 | [Data quality, quarantine, reconciliation and performance](Instructions/Labs/06-Data-Quality-Quarantine-Reconciliation-Performance.md) | Rules, rejected rows, reconciliation and Spark plans |
| 7 | [Security, Git and deployment](Instructions/Labs/07-Security-Git-CICD-and-Deployment.md) | Workspace roles, permissions, lineage, Git and release readiness |
| 8 | [End-to-end migration capstone](Instructions/Labs/08-End-to-End-Migration-Capstone.md) | Integrated execution, validation, failure recovery, cutover and rollback |

## Repository layout

- `Instructions/Labs/` – participant walkthroughs.
- `Assets/Datasets/` – synthetic source, incremental and exception files (publish/verify availability before the class).
- `Assets/Metadata/` – sample ETL configuration and watermark files (publish/verify availability before the class).
- `Assets/ReferenceNotebooks/` – representative Databricks source notebook (publish/verify availability before the class).

**Instructor note:** The lab guides refer to the asset paths above. Before distributing the repository, verify that the Assets folders and files are visible in the public GitHub repository. Until then, distribute the matching offline asset ZIP to participants. Do not assume files on an instructor's local machine are available from GitHub.

## Skillable persistence and backup

Skillable environments can be temporary. Confirm separately whether the VM disk, Fabric workspace, Lakehouse tables/files, notebooks, pipelines, Warehouse objects and OneLake data persist after an exercise ends. These are distinct persistence boundaries.

At the end of every lab, preserve notebook source, SQL scripts, pipeline definitions, configuration, validation outputs, run IDs and required screenshots in the client-approved LMS, SharePoint, OneDrive or another persistent location. GitHub is the source for starter materials; it is not a substitute for backing up Lakehouse data or every Fabric artifact.

## Compatibility and validation

Fabric navigation and available capabilities vary by tenant, capacity, permissions and Skillable image. Some steps provide an alternative when Warehouse, Git integration, deployment pipelines or Power BI authoring are not enabled.

These guides are expanded training walkthroughs, but the code and click paths must still be tested in the actual Skillable tenant before class delivery. Where the tenant behaves differently, the instructor should update the instructions and note the tested environment/version.

## Repository maintenance

See [CONTRIBUTING.md](CONTRIBUTING.md) before proposing changes. See [GITHUB_PUBLISHING_GUIDE.md](GITHUB_PUBLISHING_GUIDE.md) for publishing and verification notes.