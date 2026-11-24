# Lab 04 – Delta MERGE, SCD Type 1, SCD Type 2 and Schema Evolution

**Time:** 4 hours  
**Level:** Intermediate with beginner-friendly steps  
**Goal:** Learn deduplication, Delta MERGE, historical tracking and controlled schema changes.

## 1. Understand the two SCD types

- **SCD Type 1:** overwrite the old attribute value. The previous value is not retained.
- **SCD Type 2:** preserve history by expiring the old row and inserting a new current row with effective dates.

In this lab, Product uses Type 1 and Customer uses Type 2.

## 2. Prepare your notebook

1. Open your workspace.
2. Open or create notebook `NB_04_DELTA_SCD`.
3. Attach `LH_MFG_SILVER` as the default Lakehouse.
4. Confirm that `brz_customer` and `brz_product` are visible in the workspace's Lakehouse tables. If your Silver notebook cannot see Bronze tables directly, attach Bronze as an additional Lakehouse and use its table path/catalog as instructed by the trainer.
5. Run:

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window
from delta.tables import DeltaTable
from datetime import date

print("Spark session ready")
```

## 3. Deduplicate an incoming source

A Delta MERGE can fail if more than one source row matches the same target key. First create a deterministic source with one row per key.

```python
customer_source = spark.table("brz_customer")

w = Window.partitionBy("CustomerID").orderBy(F.col("ModifiedDate").desc())

customer_one_row = (
    customer_source
      .withColumn("rn", F.row_number().over(w))
      .filter(F.col("rn") == 1)
      .drop("rn")
)

print("Source rows:", customer_source.count())
print("Deduplicated rows:", customer_one_row.count())
```

Expected: the deduplicated count should not exceed the original count.

## 4. Implement a Type 1 Product table

1. Read the Bronze Product table.
2. Create a current-state Silver table from the baseline.
3. Merge incoming Product changes by ProductID.

```python
product_source = spark.table("brz_product")

(product_source.write.format("delta")
 .mode("overwrite").saveAsTable("slv_product"))

product_target = DeltaTable.forName(spark, "slv_product")

(product_target.alias("t")
 .merge(product_source.alias("s"), "t.ProductID = s.ProductID")
 .whenMatchedUpdateAll()
 .whenNotMatchedInsertAll()
 .execute())

display(spark.table("slv_product").limit(10))
```

This first merge is intentionally idempotent. A later update will be applied using the incremental Product JSON file.

## 5. Load the Product incremental JSON and inspect schema drift

1. Upload `Product_Incremental.json` into Bronze `Files/Incremental` if it is not already there.
2. Run:

```python
product_changes = spark.read.json(
    "abfss://<workspace-or-lakehouse-path>/Files/Incremental/Product_Incremental.json"
)
```

If the notebook is attached to the Bronze Lakehouse, prefer its relative path:

```python
product_changes = spark.read.json("Files/Incremental/Product_Incremental.json")
product_changes.printSchema()
display(product_changes.limit(5))
```

3. Compare the schema with `spark.table("brz_product").printSchema()`.
4. Notice the new `ProductSubCategory` field. This is schema drift: the source has changed its structure.

## 6. Apply a controlled schema change

For this classroom exercise, explicitly allow the new column and merge the changed rows:

```python
product_changes = product_changes.withColumn(
    "ModifiedDate", F.to_date("ModifiedDate")
)

(product_changes.write.format("delta")
 .mode("append")
 .option("mergeSchema", "true")
 .saveAsTable("slv_product_schema_test"))

display(spark.table("slv_product_schema_test").printSchema())
```

If `slv_product_schema_test` does not exist, this creates it using the incoming schema. To test evolution against the existing table, create the test table from the baseline first, then append the incremental rows with `mergeSchema=true`. Do not enable schema evolution globally without review.

## 7. Create a Type 2 Customer dimension

The Type 2 table uses:
- `StartDate`: when this version became active
- `EndDate`: when this version stopped being active
- `IsCurrent`: true for the current version
- `RecordHash`: detects changes to tracked attributes

First create the baseline:

```python
customer_base = (
    spark.table("brz_customer")
      .drop("BatchID", "IngestionTimestamp", "SourceSystem", "SourceFile")
      .withColumn("StartDate", F.to_date("ModifiedDate"))
      .withColumn("EndDate", F.lit(None).cast("date"))
      .withColumn("IsCurrent", F.lit(True))
      .withColumn(
          "RecordHash",
          F.sha2(F.concat_ws("||", "CustomerName", "Region", "Segment",
                             "Country", "Address"), 256)
      )
)

(customer_base.write.format("delta")
 .mode("overwrite").saveAsTable("slv_customer_scd2"))
```

## 8. Prepare the changed Customer rows

1. Upload `Customer_Incremental.csv` if needed.
2. Read the file.
3. Calculate the same hash over the same tracked columns.

```python
customer_changes = (
    spark.read.option("header", True).option("inferSchema", True)
         .csv("Files/Incremental/Customer_Incremental.csv")
         .withColumn("ModifiedDate", F.to_date("ModifiedDate"))
         .withColumn(
             "RecordHash",
             F.sha2(F.concat_ws("||", "CustomerName", "Region", "Segment",
                                "Country", "Address"), 256)
         )
)

customer_changes = (
    customer_changes
      .withColumn("StartDate", F.col("ModifiedDate"))
      .withColumn("EndDate", F.lit(None).cast("date"))
      .withColumn("IsCurrent", F.lit(True))
)
```

## 9. Expire changed current records and insert new versions

This is a teaching pattern. It uses a staged merge: expire only current rows whose hash changed, then insert changed/new versions. In production, make the operation restart-safe and protect against duplicate batch execution.

```python
current_customer = spark.table("slv_customer_scd2").filter("IsCurrent = true")

changed_keys = (
    customer_changes.alias("s")
      .join(current_customer.alias("t"), "CustomerID")
      .filter(F.col("s.RecordHash") != F.col("t.RecordHash"))
      .select("CustomerID")
      .distinct()
)

expire_source = changed_keys.withColumn("ExpireFlag", F.lit(True))
target = DeltaTable.forName(spark, "slv_customer_scd2")

(target.alias("t")
 .merge(expire_source.alias("s"),
        "t.CustomerID = s.CustomerID AND t.IsCurrent = true")
 .whenMatchedUpdate(set={
     "EndDate": "current_date()",
     "IsCurrent": "false"
 })
 .execute())

# Insert changed customers and new customers that do not currently exist.
current_after_expire = spark.table("slv_customer_scd2").filter("IsCurrent = true")
to_insert = (
    customer_changes.alias("s")
      .join(current_after_expire.alias("t"), "CustomerID", "left")
      .filter(F.col("t.CustomerID").isNull() |
              (F.col("s.RecordHash") != F.col("t.RecordHash")))
      .select("s.*")
)

(to_insert.write.format("delta").mode("append")
 .saveAsTable("slv_customer_scd2"))

display(spark.table("slv_customer_scd2").orderBy("CustomerID", "StartDate"))
```

## 10. Validate Type 2 history

Run:

```python
current_counts = (
    spark.table("slv_customer_scd2")
      .filter("IsCurrent = true")
      .groupBy("CustomerID").count()
      .filter("count > 1")
)

print("Customers with multiple current rows:", current_counts.count())
display(current_counts)
```

Expected: zero customers with more than one current row. Inspect C0001 and confirm the old and new versions are retained.

## 11. Compare Type 1 and Type 2

Write a short explanation:
- Which table keeps only the latest Product attributes?
- Which table retains Customer history?
- How can a report query only the current customer row?
- How can an auditor see the historical address?

## 12. Completion checklist

- [ ] Source was deduplicated by business key.
- [ ] Product Type 1 table created.
- [ ] New ProductSubCategory field identified.
- [ ] Schema evolution tested in a separate table.
- [ ] Customer SCD2 table created.
- [ ] Changed Customer records expired and new versions inserted.
- [ ] No CustomerID has more than one current row.

## Common errors

| Error | Fix |
|---|---|
| MERGE matched multiple source rows | Deduplicate source before merge |
| Cannot merge fields | Compare data types and column names; cast explicitly |
| Table not found | Attach the correct Lakehouse or use the correct table reference |
| All records appear changed | Ensure the hash uses identical columns and consistent null handling |
| Duplicate current records | Review the expire/insert logic and rerun from a clean baseline |

## Homework
Implement a Type 1 merge for Supplier. Then explain what additional columns and logic would be required to preserve Supplier history using Type 2.