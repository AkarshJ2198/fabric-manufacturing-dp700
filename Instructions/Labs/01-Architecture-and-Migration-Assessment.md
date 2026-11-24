# Lab 01 – Understand the Manufacturing Platform and Plan the Migration

**Time:** 3–4 hours  
**Level:** Beginner  
**Goal:** Understand the sample business, identify its data entities, map the existing Databricks components to Fabric, and document a target design.

## 1. Business scenario

A fictional manufacturer sells industrial products to customers, buys from suppliers, stores inventory at plants and manufactures products. The company currently uses Databricks for data processing and wants to modernize to Microsoft Fabric.

The training data contains these business entities:

| File | What it represents | Key columns |
|---|---|---|
| Customer.csv | Buyers of manufactured products | CustomerID |
| Product.csv | Products sold | ProductID |
| Supplier.csv | Companies supplying materials | SupplierID |
| Plant.csv | Manufacturing/storage locations | PlantID |
| SalesOrder.csv | Sales order header | OrderID |
| SalesOrderDetail.csv | Products and quantities in each order | OrderDetailID, OrderID, ProductID |
| InventoryMovement.csv | Stock movement at a plant | MovementID, ProductID, PlantID |
| Production.csv | Quantity produced at a plant | ProductionID, ProductID, PlantID |

## 2. Open the training workspace and inspect the files

1. Sign in to Fabric and open your assigned `MFG_<YourInitials>` workspace.
2. Open `LH_MFG_BRONZE`.
3. Select **Files** → **SourceData**.
4. Confirm that the eight CSV files from Lab 00 are present.
5. Open `Customer.csv` in the notebook using the code below. If you have not created a notebook, select **New** → **Notebook**, rename it `NB_01_SOURCE_EXPLORATION`, and attach `LH_MFG_BRONZE`.

```python
from pyspark.sql import functions as F

files = [
    "Customer", "Product", "Supplier", "Plant",
    "SalesOrder", "SalesOrderDetail",
    "InventoryMovement", "Production"
]

for name in files:
    df = (spark.read.option("header", True)
                    .option("inferSchema", True)
                    .csv(f"Files/SourceData/{name}.csv"))
    print(f"{name}: {df.count()} rows")
    df.printSchema()
```

6. Run the cell.
7. For each file, note the row count and the columns printed below it.
8. If a file fails, verify its spelling under `Files/SourceData`.

## 3. Understand the relationships

Use the following relationships when discussing the model:

- Customer.CustomerID → SalesOrder.CustomerID
- SalesOrder.OrderID → SalesOrderDetail.OrderID
- Product.ProductID → SalesOrderDetail.ProductID
- Product.ProductID → InventoryMovement.ProductID
- Plant.PlantID → InventoryMovement.PlantID
- Product.ProductID → Production.ProductID
- Plant.PlantID → Production.PlantID

Create a simple diagram in PowerPoint, Visio, draw.io or on paper. Put SalesOrder and SalesOrderDetail in the centre, with Customer and Product connected to them. Put InventoryMovement and Production alongside Product and Plant.

## 4. Inspect the Databricks reference notebook

1. In the GitHub repository, open `Assets/ReferenceNotebooks/databricks_customer_migration.py`.
2. Select **Raw** or download the file.
3. Open it in a text editor.
4. Search for the terms `dbutils.widgets`, `dbutils.fs`, `spark.read`, `saveAsTable` and `merge`.
5. Record each Databricks-specific operation and what it is used for.

A migration is not simply a copy-and-paste exercise. File paths, parameters, scheduling, security and supported APIs must be reviewed.

## 5. Map Databricks components to Fabric

Create a table with these columns: **Current Databricks component**, **Purpose**, **Potential Fabric target**, **Migration action**, **How to validate**, **Risk**.

Use these starter mappings:

| Databricks component | Purpose | Fabric target to assess |
|---|---|---|
| Cloud object storage / DBFS paths | Stores source and intermediate files | OneLake and Lakehouse Files |
| Delta tables | Stores transactional tables | Fabric Lakehouse Delta tables |
| PySpark notebooks | Cleans and transforms data | Fabric notebooks |
| Databricks Workflows | Schedules and orchestrates jobs | Fabric Data Factory pipelines |
| Databricks SQL | SQL analytics | Fabric Warehouse or Lakehouse SQL analytics endpoint |
| Unity Catalog | Data governance and access | Fabric workspace/item permissions and supported governance capabilities |
| Power BI | Reporting | Power BI semantic models and reports |

For each row, add one validation test. Example: for a migrated Delta table, compare row counts, schema, business keys and a business aggregate.

## 6. Understand the Medallion architecture

Create a simple three-layer diagram:

1. **Bronze:** preserve source data with ingestion metadata. Avoid applying business rules here.
2. **Silver:** clean data, remove duplicates, validate relationships and apply business transformations.
3. **Gold:** create dimensions, facts and aggregates designed for reporting.

In your diagram, draw arrows from source systems to Bronze, Bronze to Silver, Silver to Gold, and Gold to Power BI. Add Data Factory for orchestration and monitoring/audit alongside the pipeline.

## 7. Create a migration readiness checklist

Record whether the Skillable environment supports each capability. Ask the instructor to confirm anything you cannot verify.

| Capability | How to verify | Result |
|---|---|---|
| Fabric workspace | Workspace opens | Available / Not available |
| Lakehouse | Can create/open a Lakehouse | Available / Not available |
| Notebook and Spark | Run `display(spark.range(5))` | Available / Not available |
| Data Factory pipeline | Can create a pipeline | Available / Not available |
| Warehouse | Can create a Warehouse | Available / Not available |
| Power BI report | Can create a report from training data | Available / Not available |
| Git integration | Workspace settings show Git connection | Available / Not available |
| Deployment pipelines | Workspace has deployment pipeline access | Available / Not available |

## 8. Deliverables

Save these items:
- Source entity relationship diagram
- Databricks-to-Fabric mapping table
- Medallion architecture diagram
- Skillable readiness checklist
- Five migration risks and proposed mitigations

## 9. Homework

Identify five risks in moving a production Databricks workload to Fabric. Include at least one risk each for data correctness, performance/capacity, security, orchestration and reporting continuity. For each, write a mitigation and a test that would prove the mitigation works.

## Completion checklist

- [ ] I can explain what each of the eight source files contains.
- [ ] I identified the main business keys and relationships.
- [ ] I inspected the Databricks reference notebook.
- [ ] I completed the component mapping.
- [ ] I drew the target Medallion architecture.
- [ ] I recorded the Skillable capabilities and risks.