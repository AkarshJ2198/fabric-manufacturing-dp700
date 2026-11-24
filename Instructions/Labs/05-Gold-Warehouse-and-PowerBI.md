# Lab 05 – Gold Layer, Warehouse and Power BI

## Objective
Build curated analytical datasets and validate reporting outputs.

## Activities
1. Create dimensions `dim_customer`, `dim_product`, `dim_supplier`, and `dim_plant`. For current-state reporting, select the current SCD2 customer row while retaining history for historical analysis.
2. Create facts `fact_sales`, `fact_inventory`, and `fact_production`. Validate business keys and join cardinality.
3. Build a Gold sales aggregation by region, product category, year and month, calculating sales amount and quantity.
4. If Warehouse is enabled, create `WH_MFG_ANALYTICS`, expose curated Gold data through supported Fabric methods, and compare SQL aggregates with Lakehouse results. Otherwise use the Lakehouse SQL analytics endpoint and document the limitation.
5. Build a Power BI report for total sales, sales by region, top products, inventory movements, production by plant and monthly sales trend.
6. Reconcile report totals with SQL queries against Gold tables.

## Deliverables
Gold dimensions/facts, sales summary, Warehouse SQL or Lakehouse SQL evidence, Power BI report and reconciliation evidence.

## Self-paced task
Add a supplier performance KPI using available source fields and document assumptions.