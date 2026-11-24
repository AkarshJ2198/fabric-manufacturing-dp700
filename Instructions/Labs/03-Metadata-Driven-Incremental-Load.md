# Lab 03 – Build a Metadata-Driven Incremental Load

**Time:** 3–4 hours  
**Level:** Beginner to intermediate  
**Goal:** Understand watermarks, load configuration, incremental files, audit records and how a reusable pipeline is designed.

## 1. What does incremental loading mean?

A full load reads all source rows every time. An incremental load reads only rows that are new or changed since the last successful run. A **watermark** stores the last processed modification value.

In this lab, Customer data has a baseline snapshot and a separate file containing updates. The supplied metadata tells the notebook which source file and target table to use.

## 2. Upload the lab files

1. Open `LH_MFG_BRONZE` in Fabric.
2. Under **Files**, create folders named `Metadata` and `Incremental` if they do not already exist.
3. Upload `etl_config.csv` and `etl_watermark.csv` from `Assets/Metadata` into `Files/Metadata`.
4. Upload `Customer_Incremental.csv` and `SalesOrder_Incremental.json` from `Assets/Datasets` into `Files/Incremental`.
5. Confirm all four files are visible.

## 3. Create the metadata tables

1. Open your workspace and create a notebook named `NB_03_METADATA_LOAD`.
2. Attach `LH_MFG_BRONZE` as the default Lakehouse.
3. Paste and run:

```python
from pyspark.sql import functions as F
from pyspark.sql.types import *
from delta.tables import DeltaTable
import uuid

config_df = (spark.read.option("header", True)
             .option("inferSchema", True)
             .csv("Files/Metadata/etl_config.csv"))

watermark_df = (spark.read.option("header", True)
                .option("inferSchema", True)
                .csv("Files/Metadata/etl_watermark.csv"))

display(config_df)
display(watermark_df)
```

4. Confirm that the configuration contains columns `TableName`, `SourceFile`, `TargetTable`, `LoadType`, `WatermarkColumn` and `IsActive`.
5. Run:

```python
(config_df.write.format("delta").mode("overwrite")
 .saveAsTable("etl_config"))

(watermark_df.write.format("delta").mode("overwrite")
 .saveAsTable("etl_watermark"))

display(spark.table("etl_config"))
display(spark.table("etl_watermark"))
```

## 4. Understand the Customer watermark

1. Run `display(spark.table("etl_watermark").filter("TableName = 'Customer'"))`.
2. Note the current Customer watermark. The supplied baseline is 2026-11-20.
3. The incremental file contains changed Customer records dated 2026-11-22.
4. Therefore, records later than the watermark should be considered for this run.

## 5. Read and filter the incremental Customer file

Run:

```python
customer_incremental = (
    spark.read.option("header", True)
         .option("inferSchema", True)
         .csv("Files/Incremental/Customer_Incremental.csv")
)

current_watermark = (
    spark.table("etl_watermark")
         .filter(F.col("TableName") == "Customer")
         .select("LastWatermark")
         .first()["LastWatermark"]
)

customer_incremental = customer_incremental.withColumn(
    "ModifiedDate", F.to_date("ModifiedDate")
)

new_customer_rows = customer_incremental.filter(
    F.col("ModifiedDate") > F.to_date(F.lit(str(current_watermark)))
)

print("Rows in incremental file:", customer_incremental.count())
print("Rows after watermark:", new_customer_rows.count())
display(new_customer_rows.limit(10))
```

Expected: the supplied Customer incremental file has 60 rows, dated after the baseline watermark.

## 6. Remove duplicate business keys

A business key uniquely identifies a real-world entity. For Customer, the key is CustomerID. If a batch contains multiple versions of the same customer, keep the latest ModifiedDate.

```python
from pyspark.sql.window import Window

dedupe_window = Window.partitionBy("CustomerID").orderBy(
    F.col("ModifiedDate").desc()
)

customer_deduped = (
    new_customer_rows
      .withColumn("row_num", F.row_number().over(dedupe_window))
      .filter(F.col("row_num") == 1)
      .drop("row_num")
)

print("Rows after deduplication:", customer_deduped.count())
```

## 7. Merge the changes into a current-state table

This is a Type 1-style current-state merge. It updates existing customers and inserts new customers.

```python
target_name = "slv_customer_current"

if not spark.catalog.tableExists(target_name):
    (spark.table("brz_customer")
       .drop("BatchID", "IngestionTimestamp", "SourceSystem", "SourceFile")
       .write.format("delta").mode("overwrite")
       .saveAsTable(target_name))

customer_target = DeltaTable.forName(spark, target_name)

(customer_target.alias("t")
 .merge(customer_deduped.alias("s"), "t.CustomerID = s.CustomerID")
 .whenMatchedUpdateAll()
 .whenNotMatchedInsertAll()
 .execute())

display(spark.table(target_name).filter("CustomerID = 'C0001'"))
```

Expected: C0001 has the updated address from the incremental file.

## 8. Create an audit table

An audit record tells an operator what ran, when it ran and whether it succeeded.

```python
audit_schema = StructType([
    StructField("BatchID", StringType(), False),
    StructField("TableName", StringType(), False),
    StructField("StartTime", TimestampType(), True),
    StructField("EndTime", TimestampType(), True),
    StructField("RowsRead", LongType(), True),
    StructField("RowsMerged", LongType(), True),
    StructField("Status", StringType(), True)
])

audit_row = [(
    str(uuid.uuid4()), "Customer", None, None,
    customer_incremental.count(), customer_deduped.count(), "SUCCESS"
)]

audit_df = spark.createDataFrame(audit_row, audit_schema)
(audit_df.write.format("delta").mode("append").saveAsTable("etl_audit_log"))
display(spark.table("etl_audit_log"))
```

For a production solution, capture actual start/end timestamps and write a FAILURE record in an exception handler. This classroom example introduces the audit structure.

## 9. Advance the watermark only after successful processing

Run only after the merge and validation have succeeded:

```python
new_watermark = customer_incremental.agg(
    F.max("ModifiedDate").alias("max_date")
).first()["max_date"]

watermark_target = DeltaTable.forName(spark, "etl_watermark")

(watermark_target.alias("t")
 .merge(
     spark.createDataFrame([("Customer", new_watermark)], ["TableName", "LastWatermark"]).alias("s"),
     "t.TableName = s.TableName"
 )
 .whenMatchedUpdate(set={"LastWatermark": "s.LastWatermark"})
 .execute())

display(spark.table("etl_watermark").filter("TableName = 'Customer'"))
```

If processing fails, do not advance the watermark; otherwise the next run could skip unprocessed rows.

## 10. Read the JSON Lines SalesOrder file

The file is newline-delimited JSON (one JSON object per line). Run:

```python
sales_incremental = spark.read.json("Files/Incremental/SalesOrder_Incremental.json")
print("Incremental sales orders:", sales_incremental.count())
display(sales_incremental.limit(10))
```

The file contains new order records. Compare its schema with `brz_salesorder`. Discuss whether the target is append-only or should be merged by OrderID.

## 11. Build the orchestration design

In Fabric Data Factory, the pipeline experience may differ by tenant. If pipeline authoring is enabled:

1. Open the workspace and select **New** → **Data pipeline**.
2. Name it `PL_MFG_INCREMENTAL_LOAD`.
3. Add a **Notebook** activity to the canvas.
4. Select the activity and open its settings.
5. Choose `NB_03_METADATA_LOAD` (or your processing notebook).
6. Review the available base parameters and add values such as TableName = Customer and BatchID = a unique run identifier if supported in your environment.
7. Save the pipeline.
8. Select **Run** and review the activity status and output.
9. If Lookup and ForEach activities are available, discuss how the metadata table can drive multiple source tables without creating one hard-coded pipeline per table.

If the Skillable environment does not permit pipeline authoring, complete the notebook portion and draw the pipeline with Lookup → ForEach → Notebook → Audit/Watermark.

## 12. Verify and submit

- [ ] Metadata tables created.
- [ ] Customer incremental file contains 60 rows.
- [ ] Watermark filter selected records after 2026-11-20.
- [ ] Duplicate keys handled.
- [ ] Customer current-state table updated.
- [ ] Audit record created.
- [ ] Watermark advanced only after successful validation.
- [ ] SalesOrder JSON Lines file read successfully.
- [ ] Pipeline or orchestration diagram saved.

## Common errors

| Error | Fix |
|---|---|
| Cannot infer schema from JSON | Confirm the file is newline-delimited JSON and use `spark.read.json` |
| Watermark comparison type error | Convert both sides to Date using `to_date` |
| MERGE matches multiple source rows | Deduplicate source by business key before MERGE |
| Table not found | Complete Lab 02 and verify the Bronze table exists |
| Watermark moved despite failure | Restore the previous value and only update after successful validation |

## Homework
Add Product as another metadata-driven source. Use `Product_Incremental.json` and explain how the new ProductSubCategory column should be handled.