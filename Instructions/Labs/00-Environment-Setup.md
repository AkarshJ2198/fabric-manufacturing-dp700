# Lab 00 – Environment Setup and Getting the Assets

## Objective
Prepare an individual Microsoft Fabric workspace and retrieve the training assets from the course GitHub repository.

## Prerequisites
- Skillable DP-700 lab access
- Fabric workspace access
- Lakehouse and notebook creation permissions
- Browser access to GitHub

## Step 1 – Open the course repository
1. Open the course repository URL shared by the trainer.
2. Select **Code** → **Download ZIP**, or open the `Assets` folder and download the required files.
3. Extract the ZIP locally if using Download ZIP.
4. Do not upload the entire repository ZIP into a Lakehouse.

## Step 2 – Create the workspace
1. Open the Fabric portal.
2. Select the assigned training workspace, if one is pre-created.
3. Otherwise create a workspace named `MFG_<YourInitials>`.
4. Confirm that the workspace has an eligible Fabric capacity.

## Step 3 – Create the Lakehouses
Create `LH_MFG_BRONZE`, `LH_MFG_SILVER`, and `LH_MFG_GOLD`. Create `WH_MFG_ANALYTICS` only if Warehouse is enabled for the lab.

## Step 4 – Upload source files
1. Open `LH_MFG_BRONZE` → **Files**.
2. Create a folder named `SourceData`.
3. Upload only the files required by the current lab from `Assets/Datasets`.
4. Preserve the original filenames.

## Step 5 – Validate access
Create a Fabric notebook, attach `LH_MFG_BRONZE`, and run:

```python
display(spark.range(5))
```

Confirm that five rows are displayed.

## Completion checklist
- [ ] Fabric workspace opens
- [ ] Bronze, Silver and Gold Lakehouses are created
- [ ] Source files are visible under Bronze Files/SourceData
- [ ] Notebook can execute Spark code

## Troubleshooting
- If Lakehouse creation is unavailable, check workspace capacity and permissions with the lab administrator.
- If GitHub download is blocked, ask the trainer for the approved offline asset bundle.
- If the workspace is reset between sessions, repeat this setup using the daily baseline instructions.