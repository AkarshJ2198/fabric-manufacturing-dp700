# Lab 00 – Set Up Your Fabric Environment (Beginner Walkthrough)

**Time:** 60–90 minutes  
**Goal:** Sign in, locate your Skillable environment, create or identify the Fabric workspace and Lakehouses, download the course data, upload it to OneLake, and run your first notebook.

> **Important:** Fabric screens and available buttons can differ by tenant, capacity and Skillable lab version. Follow the trainer's assigned workspace and capacity instructions whenever they differ from this guide. Do not create extra workspaces or capacities unless instructed.

## 1. What you are setting up

You will work with three Lakehouses:

| Item | Purpose |
|---|---|
| `LH_MFG_BRONZE` | Raw source files and minimally changed source data |
| `LH_MFG_SILVER` | Cleaned, deduplicated and conformed data |
| `LH_MFG_GOLD` | Business-ready tables for analytics |

A Lakehouse stores files and Delta tables in OneLake. A notebook is where you will use PySpark to read, transform and write data.

## 2. Start the Skillable lab

1. Open the Skillable lab link supplied by your trainer.
2. Sign in using the credentials supplied for this class. Do not use your personal Microsoft account unless the trainer specifically tells you to.
3. Wait until the lab status indicates that the environment is ready.
4. Open the lab instructions pane and note the assigned username, tenant and any workspace name. Never paste passwords into GitHub or notebooks.
5. In the lab desktop, open Microsoft Edge.
6. Navigate to **https://app.fabric.microsoft.com**.
7. Sign in with the training account. If a tenant selector appears, choose the tenant specified by the trainer.
8. If Fabric displays a capacity, permission or license error, stop and show the exact error to the trainer. Do not attempt to purchase or assign capacity.

## 3. Identify your workspace

1. In Fabric, select **Workspaces** from the left navigation.
2. Look for a workspace allocated to your Skillable user. The trainer may have created one in advance.
3. Select that workspace and confirm that its contents load.
4. If no workspace is assigned and you have permission to create one, select **New workspace**.
5. Enter `MFG_<YourInitials>` as the name, replacing `<YourInitials>` with your initials. Example: `MFG_AK`.
6. Leave advanced settings at their defaults unless the trainer provides specific values.
7. Select **Apply** or **Create**.
8. Confirm that the new workspace appears in the workspace list.

**If you cannot create a workspace:** use the assigned workspace or ask the trainer. Workspace creation can be restricted by tenant policy.

## 4. Create the Bronze, Silver and Gold Lakehouses

Repeat these steps for each Lakehouse:

1. Open your training workspace.
2. Select **New** (or **+ New item**) in the workspace toolbar.
3. Find and select **Lakehouse**.
4. Enter the exact name `LH_MFG_BRONZE`.
5. Select **Create** and wait for the Lakehouse explorer to open.
6. Return to the workspace item list.
7. Repeat the process for `LH_MFG_SILVER` and `LH_MFG_GOLD`.

Your workspace should now contain three Lakehouse items. Do not create a Warehouse for this lab unless your trainer has confirmed it is enabled.

## 5. Download the course files from GitHub

1. Open the public repository: https://github.com/AkarshJ2198/fabric-manufacturing-dp700
2. Select the green **Code** button.
3. Select **Download ZIP**.
4. Wait for the ZIP to download.
5. In File Explorer, open **Downloads**.
6. Right-click the downloaded ZIP and select **Extract All**.
7. Open the extracted folder.
8. Navigate to `Assets/Datasets` and `Assets/Metadata`.

You should see files such as `Customer.csv`, `Product.csv`, `SalesOrder.csv`, `InventoryMovement.csv`, `etl_config.csv` and `etl_watermark.csv`. If the files are not present in the repository yet, use the offline bundle provided by the trainer.

## 6. Upload the source files to the Bronze Lakehouse

1. In Fabric, open `LH_MFG_BRONZE`.
2. In the Lakehouse explorer, locate **Files**.
3. Select the ellipsis next to **Files** (or use the **New folder** option).
4. Create a folder named `SourceData`.
5. Select the `SourceData` folder so that it is open.
6. Select **Upload** → **Upload files**.
7. Browse to the extracted `Assets/Datasets` folder.
8. Select the eight baseline files:
   - `Customer.csv`
   - `Product.csv`
   - `Supplier.csv`
   - `Plant.csv`
   - `SalesOrder.csv`
   - `SalesOrderDetail.csv`
   - `InventoryMovement.csv`
   - `Production.csv`
9. Select **Open** and wait for all uploads to complete.
10. Confirm that the filenames appear under `Files/SourceData`.

Do not upload the ZIP file itself. Upload the extracted data files.

## 7. Create and attach your first notebook

1. Return to the workspace item list.
2. Select **New** → **Notebook**.
3. Rename the notebook to `NB_00_ENVIRONMENT_CHECK`.
4. In the notebook toolbar, find **Add lakehouse** or the Lakehouse attachment area.
5. Select **Add lakehouse**.
6. Choose **Existing lakehouse** and select `LH_MFG_BRONZE`.
7. Confirm that the Lakehouse appears in the notebook explorer. If it is not the default Lakehouse, use the notebook's available option to make it the default.

## 8. Run your first Spark cell

1. Select the first empty code cell.
2. Paste the following code:

```python
display(spark.range(5))
```

3. Select the cell's **Run** icon, or press **Shift + Enter**.
4. Wait for the cell to finish.
5. Confirm that a result table with five rows appears, containing values 0 through 4.

If the cell is still running, wait. If it fails, check that a Lakehouse is attached and that your workspace has an available Fabric capacity.

## 9. Check that your files can be read

Create a new cell and run:

```python
from pyspark.sql import functions as F

customer_df = (
    spark.read
         .option("header", True)
         .option("inferSchema", True)
         .csv("Files/SourceData/Customer.csv")
)

print("Customer rows:", customer_df.count())
display(customer_df.limit(10))
```

Expected result: the baseline Customer file contains **500 rows**. The preview should show columns such as CustomerID, CustomerName, Region, Segment, Country, Address and ModifiedDate.

If you get a file-not-found error, open the Lakehouse explorer and verify the exact folder and filename. File paths are case-sensitive in many Spark contexts.

## 10. Save your work and record the environment

1. Save the notebook.
2. Record your workspace name, Lakehouse names, notebook name and date in your personal lab notes.
3. Take a screenshot showing the successful Spark result and Customer row count.
4. Save the screenshot and notebook source to the persistent location specified by the trainer.

## 11. End-of-lab checklist

- [ ] I can open the assigned Fabric workspace.
- [ ] I can see `LH_MFG_BRONZE`, `LH_MFG_SILVER` and `LH_MFG_GOLD`.
- [ ] The eight baseline CSV files are under Bronze `Files/SourceData`.
- [ ] My notebook is attached to `LH_MFG_BRONZE`.
- [ ] `spark.range(5)` ran successfully.
- [ ] I read `Customer.csv` and confirmed 500 rows.
- [ ] I saved my notebook and evidence outside the temporary VM.

## Common problems

| Problem | What to check |
|---|---|
| Fabric sign-in loop | Use the Skillable training credentials and correct tenant |
| Workspace creation disabled | Use the trainer-assigned workspace |
| Lakehouse option missing | Check workspace permissions and capacity with trainer |
| Notebook cell cannot start | Confirm a Lakehouse is attached and capacity is running |
| File not found | Check the folder is `Files/SourceData` and filename spelling |
| CSV count is zero | Confirm the correct file was uploaded and header option is enabled |

## Instructor checkpoint
Do not proceed to Lab 01 until every checklist item above is complete.