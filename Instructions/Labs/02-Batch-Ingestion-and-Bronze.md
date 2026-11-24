# Lab 02 – Batch Ingestion and Bronze Layer

## Objective
Ingest manufacturing source files into OneLake and create raw Bronze Delta tables.

## Files required
`Customer.csv`, `Product.csv`, `Supplier.csv`, `Plant.csv`, `SalesOrder.csv`, `SalesOrderDetail.csv`, `InventoryMovement.csv`, and `Production.csv` from `Assets/Datasets`.

## Activities
1. Upload the eight CSV files to `LH_MFG_BRONZE/Files/SourceData`.
2. Create notebook `NB_01_BRONZE_INGESTION` and attach `LH_MFG_BRONZE`.
3. Read Customer.csv:

```python
from pyspark.sql import functions as F
import uuid

df = (spark.read.option("header", True).option("inferSchema", True)
      .csv("Files/SourceData/Customer.csv"))
display(df.limit(10))
```

4. Add ingestion metadata:

```python
batch_id = str(uuid.uuid4())
df_bronze = (df.withColumn("BatchID", F.lit(batch_id))
               .withColumn("IngestionTimestamp", F.current_timestamp())
               .withColumn("SourceSystem", F.lit("ERP")))
df_bronze.write.format("delta").mode("overwrite").saveAsTable("brz_customer")
```

5. Repeat for the other files using `brz_product`, `brz_supplier`, `brz_plant`, `brz_salesorder`, `brz_salesorderdetail`, `brz_inventorymovement`, and `brz_production`.
6. Count and record each table.

## Expected baseline counts
Customer 500; Product 100; Supplier 60; Plant 4; SalesOrder 10,000; InventoryMovement 5,000; Production 2,500. SalesOrderDetail count should be read from the supplied file.

## Deliverables
Eight Bronze Delta tables, ingestion notebook and count validation output.

## Self-paced task
Ingest a second copy of Product under a separate batch ID and explain why Bronze should preserve source history rather than silently overwrite it.