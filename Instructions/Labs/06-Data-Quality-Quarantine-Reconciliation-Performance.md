# Lab 06 – Data Quality, Quarantine, Reconciliation and Performance

**Time:** 3–4 hours  
**Level:** Intermediate  
**Goal:** Detect bad data, separate valid and rejected records, reconcile counts and inspect Spark execution.

## 1. What you will build

- A set of data quality rules
- A valid records output
- A quarantine table for rejected rows
- A reconciliation result
- A simple audit record

## 2. Upload exception files

1. Open `LH_MFG_BRONZE`.
2. Under **Files**, create a folder named `Exceptions`.
3. Upload `Customer_Exceptions.csv` and `Sales_Exceptions.csv` from the repository's `Assets/Datasets` folder.
4. Confirm both files are visible.

## 3. Create the quality notebook

1. Open your workspace.
2. Select **New** → **Notebook**.
3. Name it `NB_06_DATA_QUALITY`.
4. Attach `LH_MFG_BRONZE` as the default Lakehouse.
5. Run:

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window
from pyspark.sql.types import *
import uuid

batch_id = str(uuid.uuid4())
```

## 4. Read the exception datasets

```python
customer_bad_source = (
    spark.read.option("header", True).option("inferSchema", True)
         .csv("Files/Exceptions/Customer_Exceptions.csv")
)

sales_bad_source = (
    spark.read.option("header", True).option("inferSchema", True)
         .csv("Files/Exceptions/Sales_Exceptions.csv")
)

print("Customer exception input:", customer_bad_source.count())
print("Sales exception input:", sales_bad_source.count())
display(customer_bad_source)
display(sales_bad_source.limit(10))
```

Expected: 3 Customer exception rows and 100 Sales exception rows.

## 5. Apply Customer validation rules

Rules:
1. CustomerID must not be null or blank.
2. CustomerName must not be null or blank.
3. CustomerID must not already exist in the baseline Customer table.

```python
existing_customer_ids = (
    spark.table("brz_customer")
         .select("CustomerID").distinct()
)

customer_checked = (
    customer_bad_source
      .withColumn(
          "RejectReason",
          F.when(F.col("CustomerID").isNull() | (F.trim("CustomerID") == ""),
                 "CustomerID is missing")
           .when(F.col("CustomerName").isNull() | (F.trim("CustomerName") == ""),
                 "CustomerName is missing")
           .when(F.col("CustomerID").isin(
               [r["CustomerID"] for r in existing_customer_ids.collect()]
           ), "CustomerID already exists in baseline")
           .otherwise(F.lit(None).cast("string"))
      )
      .withColumn("BatchID", F.lit(batch_id))
      .withColumn("ValidationTimestamp", F.current_timestamp())
)

customer_valid = customer_checked.filter(F.col("RejectReason").isNull())
customer_quarantine = customer_checked.filter(F.col("RejectReason").isNotNull())

print("Valid Customer rows:", customer_valid.count())
print("Quarantined Customer rows:", customer_quarantine.count())
display(customer_quarantine)
```

Expected: the duplicate C0001 and the missing-key row are rejected. C9998 has a missing name and is rejected. The valid count is zero for this supplied exception file.

## 6. Validate Sales exception rows

Rules:
- OrderID must be populated.
- CustomerID must exist in the Customer dimension/source.
- ProductID must exist in Product.
- Quantity must be greater than zero.
- UnitPrice must be non-negative.

```python
valid_customer_ids = spark.table("brz_customer").select("CustomerID").distinct()
valid_product_ids = spark.table("brz_product").select("ProductID").distinct()

sales_checked = (
    sales_bad_source
      .join(valid_customer_ids.withColumn("CustomerExists", F.lit(True)),
            "CustomerID", "left")
      .join(valid_product_ids.withColumn("ProductExists", F.lit(True)),
            "ProductID", "left")
      .withColumn(
          "RejectReason",
          F.when(F.col("OrderID").isNull() | (F.trim("OrderID") == ""),
                 "OrderID is missing")
           .when(F.col("CustomerExists").isNull(), "CustomerID not found")
           .when(F.col("ProductExists").isNull(), "ProductID not found")
           .when(F.col("Quantity").isNull() | (F.col("Quantity") <= 0),
                 "Quantity must be greater than zero")
           .when(F.col("UnitPrice").isNull() | (F.col("UnitPrice") < 0),
                 "UnitPrice must be zero or greater")
           .otherwise(F.lit(None).cast("string"))
      )
      .withColumn("BatchID", F.lit(batch_id))
      .withColumn("ValidationTimestamp", F.current_timestamp())
      .drop("CustomerExists", "ProductExists")
)

sales_valid = sales_checked.filter(F.col("RejectReason").isNull())
sales_quarantine = sales_checked.filter(F.col("RejectReason").isNotNull())

print("Valid Sales rows:", sales_valid.count())
print("Quarantined Sales rows:", sales_quarantine.count())
display(sales_quarantine.groupBy("RejectReason").count())
```

## 7. Save valid and rejected records

```python
(customer_valid.write.format("delta").mode("overwrite")
 .saveAsTable("dq_customer_valid"))

(customer_quarantine.write.format("delta").mode("overwrite")
 .saveAsTable("dq_customer_quarantine"))

(sales_valid.write.format("delta").mode("overwrite")
 .saveAsTable("dq_sales_valid"))

(sales_quarantine.write.format("delta").mode("overwrite")
 .saveAsTable("dq_sales_quarantine"))
```

For a recurring production process, append quarantine records by batch rather than overwriting the table.

## 8. Reconcile counts

A complete classification should satisfy:

**Source rows = valid rows + quarantined rows**

```python
reconciliation_rows = [
    ("Customer", customer_bad_source.count(),
     customer_valid.count(), customer_quarantine.count()),
    ("Sales", sales_bad_source.count(),
     sales_valid.count(), sales_quarantine.count())
]

recon = spark.createDataFrame(
    reconciliation_rows,
    ["Entity", "SourceCount", "ValidCount", "QuarantineCount"]
).withColumn(
    "Difference",
    F.col("SourceCount") - F.col("ValidCount") - F.col("QuarantineCount")
).withColumn(
    "Status",
    F.when(F.col("Difference") == 0, "PASS").otherwise("FAIL")
)

display(recon)
(recon.write.format("delta").mode("overwrite")
 .saveAsTable("dq_reconciliation"))
```

Expected: Difference = 0 and Status = PASS for both entities.

## 9. Inspect Spark execution

1. Create a new cell and run:

```python
sales_checked.explain("formatted")
```

2. Review the physical plan output.
3. Discuss which operations are filters, joins and projections.
4. Explain why large joins can cause shuffles and why filtering early can reduce data movement.
5. Open the notebook run details and inspect duration and Spark resource usage if the Skillable environment exposes them.

Do not run optimization commands against shared or production data. Capacity and Delta optimization features may be restricted in Skillable.

## 10. Completion checklist

- [ ] Exception files uploaded.
- [ ] Customer and Sales rules applied.
- [ ] Valid and rejected rows separated.
- [ ] Quarantine tables created.
- [ ] Reconciliation difference is zero.
- [ ] Spark execution plan inspected.
- [ ] Evidence and notebook saved outside the VM.

## Homework
Add a rule that rejects inactive suppliers for a chosen transaction type. Explain whether it is a row-level validation, a referential validation or a business rule.