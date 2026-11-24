# Lab 05 – Build Gold Tables and a Power BI Report

**Time:** 3–4 hours  
**Level:** Beginner to intermediate  
**Goal:** Create business-friendly tables and a report for manufacturing sales, inventory and production.

## 1. Understand the Gold layer

Gold contains data shaped for business questions. In this lab you will create:
- Dimensions: Customer, Product, Supplier and Plant
- Facts: Sales, Inventory and Production
- A sales summary for reporting

A dimension describes an entity. A fact records an event or measurable activity.

## 2. Prepare your notebook and tables

1. Open the workspace.
2. Create a notebook named `NB_05_GOLD_MODEL`.
3. Attach `LH_MFG_GOLD` as the default Lakehouse.
4. Attach Bronze and Silver Lakehouses as additional Lakehouses if the workspace supports it.
5. Confirm the required source tables exist. Complete Labs 02–04 first.

If cross-Lakehouse table references are not available in your Skillable setup, create the Gold Lakehouse notebook with the relevant Lakehouse attached and use the OneLake paths supplied by the trainer.

## 3. Create the Customer dimension

Run:

```python
customer_dim = spark.table("slv_customer_scd2").filter("IsCurrent = true")

(customer_dim.select(
    "CustomerID", "CustomerName", "Region", "Segment", "Country"
 ).write.format("delta").mode("overwrite").saveAsTable("dim_customer"))
```

Verify:

```python
print("Customer dimension rows:", spark.table("dim_customer").count())
display(spark.table("dim_customer").limit(10))
```

## 4. Create Product, Supplier and Plant dimensions

Run:

```python
product_dim = spark.table("slv_product")
supplier_dim = spark.table("brz_supplier")
plant_dim = spark.table("brz_plant")

(product_dim.write.format("delta").mode("overwrite").saveAsTable("dim_product"))
(supplier_dim.write.format("delta").mode("overwrite").saveAsTable("dim_supplier"))
(plant_dim.write.format("delta").mode("overwrite").saveAsTable("dim_plant"))
```

If a source table is in a different Lakehouse, use the fully qualified table name/path visible in your notebook explorer.

## 5. Create the Sales fact table

SalesOrder is the header; SalesOrderDetail contains product and quantity details. Join them by OrderID.

```python
orders = spark.table("brz_salesorder")
details = spark.table("brz_salesorderdetail")

sales_fact = (
    details.alias("d")
      .join(orders.alias("o"), "OrderID", "inner")
      .withColumn("SalesAmount", F.col("Quantity") * F.col("UnitPrice"))
      .select(
          "OrderDetailID", "OrderID", "CustomerID", "ProductID",
          "OrderDate", "Status", "Quantity", "UnitPrice", "SalesAmount"
      )
)

(sales_fact.write.format("delta")
 .mode("overwrite").saveAsTable("fact_sales"))

display(spark.table("fact_sales").limit(10))
```

## 6. Create Inventory and Production facts

```python
inventory_fact = spark.table("brz_inventorymovement")
production_fact = spark.table("brz_production")

(inventory_fact.write.format("delta")
 .mode("overwrite").saveAsTable("fact_inventory"))

(production_fact.write.format("delta")
 .mode("overwrite").saveAsTable("fact_production"))
```

## 7. Create a monthly sales summary

```python
sales_summary = (
    spark.table("fact_sales")
      .withColumn("OrderDate", F.to_date("OrderDate"))
      .withColumn("SalesYear", F.year("OrderDate"))
      .withColumn("SalesMonth", F.month("OrderDate"))
      .groupBy("SalesYear", "SalesMonth")
      .agg(
          F.sum("SalesAmount").alias("TotalSales"),
          F.sum("Quantity").alias("TotalQuantity"),
          F.countDistinct("OrderID").alias("OrderCount")
      )
      .orderBy("SalesYear", "SalesMonth")
)

(sales_summary.write.format("delta")
 .mode("overwrite").saveAsTable("gold_monthly_sales"))

display(spark.table("gold_monthly_sales"))
```

## 8. Validate the Gold model

Run these checks:

```python
print("Sales fact rows:", spark.table("fact_sales").count())
print("Inventory fact rows:", spark.table("fact_inventory").count())
print("Production fact rows:", spark.table("fact_production").count())

print("Sales amount:", spark.table("fact_sales")
      .agg(F.sum("SalesAmount")).first()[0])
```

Compare the Sales fact row count with the number of valid SalesOrderDetail rows. Differences can occur if orders do not match headers; investigate rather than silently ignoring them.

## 9. Create a Power BI report

1. In the Fabric workspace, locate the Gold Lakehouse.
2. Open its SQL analytics endpoint if enabled, or use the Warehouse if your trainer has provisioned one.
3. Choose the option to create a Power BI report or semantic model from the available Fabric item.
4. Select the Gold tables needed for the report: `fact_sales`, `dim_customer`, `dim_product`, `fact_inventory` and `fact_production`.
5. In the model view, create relationships using the business keys:
   - Customer dimension to Sales fact by CustomerID
   - Product dimension to Sales fact by ProductID
   - Product dimension to Inventory fact by ProductID
   - Plant dimension to Inventory and Production facts by PlantID
6. Confirm that the relationship direction and cardinality make sense. Dimensions should normally filter facts.
7. Create a report page with:
   - Card: Total Sales
   - Bar chart: Sales by Region
   - Bar chart: Top 10 Products by Sales
   - Line chart: Monthly Sales
   - Column chart: Production Quantity by Plant
   - Table: Inventory Movement by Plant and Product
8. Save the report as `RPT_MFG_OVERVIEW`.

If Power BI report authoring is not enabled in the Skillable tenant, create the Gold tables and submit screenshots of their SQL/PySpark aggregates. The trainer can demonstrate the report in a prepared workspace.

## 10. Completion checklist

- [ ] Four dimensions created.
- [ ] Three fact tables created.
- [ ] Monthly sales aggregate created.
- [ ] Sales fact count and total sales recorded.
- [ ] Power BI report created or the tenant limitation documented.
- [ ] Report totals reconciled with the Gold tables.

## Homework
Create a supplier KPI. State the business definition clearly, identify the source columns used and explain any assumptions.