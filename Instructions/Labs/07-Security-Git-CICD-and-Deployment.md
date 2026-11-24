# Lab 07 – Security, Governance, Git and Deployment

**Time:** 3–4 hours  
**Level:** Beginner to intermediate  
**Goal:** Understand how Fabric assets are secured, tracked and promoted between environments.

> Skillable permissions vary. This lab is partly a guided inspection exercise. Do not change tenant-wide settings or grant access to real users.

## 1. Understand workspace roles

Fabric workspace roles commonly include:
- **Admin:** manages workspace settings and access.
- **Member:** can publish and manage workspace content with broad permissions.
- **Contributor:** can create and edit content.
- **Viewer:** can view content but has limited authoring ability.

The exact capabilities can depend on the item and tenant configuration.

## 2. Inspect workspace access

1. Open the Fabric workspace.
2. Select **Manage access** or the workspace access option in the toolbar.
3. Review the list of users/groups and their assigned roles.
4. Do not add or remove anyone during this exercise.
5. Record which role your Skillable user has.
6. Explain which role should be assigned to a data engineer who builds pipelines and notebooks, and which role is suitable for a report consumer.

## 3. Inspect item-level security and lineage

1. Open `LH_MFG_GOLD`.
2. Locate the available permissions or **Manage permissions** option.
3. Review the permissions shown; do not change them.
4. Return to the workspace item list.
5. Open the lineage view if available.
6. Identify the upstream Lakehouse/notebook/pipeline and downstream report dependencies.
7. Take a screenshot of the lineage view or record why it is unavailable.

## 4. Understand Git integration

The course GitHub repository contains the lab instructions and starter assets. Fabric workspace Git integration is a separate feature that can synchronize supported Fabric workspace items with a Git provider. It does not automatically mean every OneLake file or table is source-controlled.

If Git integration is enabled:
1. Open the workspace.
2. Open **Workspace settings**.
3. Find **Git integration**.
4. Review the provider and connection status.
5. Only connect a repository if the trainer has supplied an approved repository and branch.
6. Review the supported item list before committing or updating.
7. Do not connect a personal production repository to the training workspace.

If Git integration is disabled:
1. Open a notebook.
2. Use the available export/download option to save its source.
3. Save SQL scripts and configuration files in a local course folder.
4. Explain how these files would be committed to a Git repository.
5. Do not upload credentials or connection secrets.

## 5. Understand Dev/Test/Prod promotion

A common release pattern is:
- **Development:** engineers build and test changes.
- **Test:** changes are validated with representative data and business users.
- **Production:** approved release used by business consumers.

If deployment pipelines are enabled:
1. Open the deployment pipelines experience from Fabric.
2. Review the available stages.
3. Do not deploy to a real production workspace.
4. Inspect how a workspace is assigned to a stage.
5. Discuss environment-specific connections, parameters and permissions.

If deployment pipelines are not enabled, draw the Dev → Test → Prod flow and list the manual approvals and checks required at each stage.

## 6. Create a release checklist

Create a table with these columns: Item, Owner, Validation, Approval, Status. Include:
- Notebook code reviewed
- Pipeline parameters checked
- Data quality tests passed
- Source-to-target reconciliation passed
- Permissions reviewed
- Power BI report validated
- Rollback version available
- Business owner approval received

## 7. Completion checklist

- [ ] Workspace role identified.
- [ ] Least-privilege roles explained.
- [ ] Lakehouse permissions inspected.
- [ ] Lineage inspected or limitation documented.
- [ ] Git integration reviewed or source export completed.
- [ ] Dev/Test/Prod deployment approach documented.
- [ ] Release checklist completed.

## Homework
Design a role matrix for data engineers, data analysts, report consumers and workspace administrators. Explain why report consumers should not automatically receive Contributor access.