# Lab 02 – Load Manufacturing Files into the Bronze Lakehouse

**Time:** 3–4 hours  
**Level:** Beginner  
**Goal:** Upload source files, read them with PySpark, add ingestion metadata and create Bronze Delta tables.

## What you will build

You will create eight Bronze tables: `brz_customer`, `brz_product`, `brz_supplier`, `brz_plant`, `brz_salesorder`, `brz_salesorderdetail`, `brz_inventorymovement` and `brz_production`.

## 1. Confirm the files are uploaded

1. Open your Fabric workspace.
2. Open `LH_MFG_BRONZE`.
3. Select **Files** → **SourceData**.
4. Confirm the eight CSV files are present.
5. If any file is missing, use **Upload** → **Upload files** to upload it from the extracted GitHub assets folder.

## 2. Create the ingestion notebook

1. Return to the workspace.
2. Select **New** → **Notebook**.
3. Rename it `NB_02_BRONZE_INGESTION`.
4. Attach `LH_MFG_BRONZE` as the default Lakehouse.
5. Confirm that the notebook explorer shows the Lakehouse and its Files/Tables sections.

## 3. Read the Customer source file

1. In the first code cell, paste the following code.
2. Select **Run**.
3. Review the schema and first ten records.

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window
from datetime import datetime
import uuid

customer_raw = (
    spark.read
         .option("header", True)
         .option("inferSchema", True)
         .csv("Files/SourceData/Customer.csv")
)

print("Rows:", customer_raw.count())
customer_raw.printSchema()
display(customer_raw.limit(10))
```

Expected: 500 records. The columns should include CustomerID, CustomerName, Region, Segment, Country, Address and ModifiedDate.

## 4. Add ingestion metadata

In the next cell, paste and run:

```python
batch_id = str(uuid.uuid4())

customer_bronze = (
    customer_raw
      .withColumn("BatchID", F.lit(batch_id))
      .withColumn("IngestionTimestamp", F.current_timestamp())
      .withColumn("SourceSystem", F.lit("ERP"))
      .withColumn("SourceFile", F.lit("Customer.csv"))
)

display(customer_bronze.limit(10))
```

What these new columns mean:
- `BatchID`: unique identifier for this execution.
- `IngestionTimestamp`: time the data was loaded.
- `SourceSystem`: source label for lineage.
- `SourceFile`: original filename.

## 5. Write the first Bronze Delta table

Run:

```python
(customer_bronze.write
    .format("delta")
    .mode("overwrite")
    .saveAsTable("brz_customer"))
```

The first time you run this cell, Spark creates the table. `overwrite` is used here because this is a classroom baseline load. In a production ingestion design, decide explicitly whether to append, overwrite or merge; do not blindly overwrite historical Bronze data.

## 6. Verify the table was created

Run:

```python
display(spark.table("brz_customer").limit(10))
print("Bronze customer rows:", spark.table("brz_customer").count())
```

Expected: 500 rows. In the Lakehouse explorer, refresh **Tables** and look for `brz_customer`. If it does not appear immediately, refresh the explorer.

## 7. Load the remaining seven files

Run the following cell. It reads each CSV, adds the same audit columns and writes a Bronze Delta table.

```python
source_to_table = {
    "Product": "brz_product",
    "Supplier": "brz_supplier",
    "Plant": "brz_plant",
    "SalesOrder": "brz_salesorder",
    "SalesOrderDetail": "brz_salesorderdetail",
    "InventoryMovement": "brz_inventorymovement",
    "Production": "brz_production"
}

load_results = []

for source_name, target_table in source_to_table.items():
    source_path = f"Files/SourceData/{source_name}.csv"
    df = (spark.read.option("header", True)
                    .option("inferSchema", True)
                    .csv(source_path))

    current_batch_id = str(uuid.uuid4())
    df_with_audit = (
        df.withColumn("BatchID", F.lit(current_batch_id))
          .withColumn("IngestionTimestamp", F.current_timestamp())
          .withColumn("SourceSystem", F.lit("ERP"))
          .withColumn("SourceFile", F.lit(f"{source_name}.csv"))
    )

    (df_with_audit.write
        .format("delta")
        .mode("overwrite")
        .saveAsTable(target_table))

    row_count = spark.table(target_table).count()
    load_results.append((source_name, target_table, row_count, current_batch_id))

load_summary = spark.createDataFrame(
    load_results,
    ["SourceName", "TargetTable", "RowsLoaded", "BatchID"]
)
display(load_summary)
```

Expected baseline counts:
- Product: 100
- Supplier: 60
- Plant: 4
- SalesOrder: 10,000
- InventoryMovement: 5,000
- Production: 2,500
- SalesOrderDetail: count the supplied file; do not assume a count.

## 8. Verify all eight tables

Run:

```python
tables_to_check = [
    "brz_customer", "brz_product", "brz_supplier", "brz_plant",
    "brz_salesorder", "brz_salesorderdetail",
    "brz_inventorymovement", "brz_production"
]

verification = []
for table_name in tables_to_check:
    count_value = spark.table(table_name).count()
    verification.append((table_name, count_value))

display(spark.createDataFrame(verification, ["TableName", "RowCount"]))
```

Compare the results with the expected counts above. If a count is lower, check that the source file was uploaded completely and that the correct table was written.

## 9. Explore the Bronze data

Run the following query to see how many records were loaded per source file:

```python
display(
    spark.table("brz_customer")
         .groupBy("SourceFile", "SourceSystem")
         .count()
)
```

Then inspect the schema:

```python
spark.table("brz_customer").printSchema()
```

## 10. Save your notebook and evidence

1. Save the notebook.
2. Take a screenshot of the load summary and verification counts.
3. Export or copy the notebook source to the persistent location specified by the trainer.
4. Record the notebook name, table names, row counts and batch IDs.

## Common errors and fixes

| Error | Likely reason | Fix |
|---|---|---|
| Path does not exist | File not uploaded or wrong folder | Check `Files/SourceData` and exact filename |
| Table already exists | Cell was run earlier | Baseline lab uses overwrite; confirm correct table before rerun |
| No default Lakehouse | Notebook is not attached | Add `LH_MFG_BRONZE` and make it default |
| Spark session unavailable | Capacity or session issue | Check capacity and restart session if permitted |
| Schema looks wrong | Header/inference issue | Confirm first line is the CSV header and `header=True` |

## Homework

1. Explain the difference between a file in the Lakehouse **Files** area and a Delta table in **Tables**.
2. Explain why the Bronze table includes BatchID and IngestionTimestamp.
3. Propose how you would preserve multiple source batches rather than overwrite the Bronze table each time.

## Completion checklist

- [ ] Eight source files are available in Bronze Files.
- [ ] Eight Bronze Delta tables were created.
- [ ] Customer count is 500.
- [ ] Product count is 100.
- [ ] Supplier count is 60.
- [ ] Plant count is 4.
- [ ] SalesOrder count is 10,000.
- [ ] InventoryMovement count is 5,000.
- [ ] Production count is 2,500.
- [ ] SalesOrderDetail count was recorded from the actual file.
- [ ] Notebook and evidence were saved outside the temporary VM.