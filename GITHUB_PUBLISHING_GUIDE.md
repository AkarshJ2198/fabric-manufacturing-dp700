# GitHub publishing guide

Target repository: `AkarshJ2198/fabric-manufacturing-dp700`
Visibility: Public

The repository has been created. To publish or update the complete local starter pack, extract the ZIP and run from its root:

```bash
git init
git branch -M main
git add .
git commit -m "Publish manufacturing DP-700 training labs"
git remote add origin https://github.com/AkarshJ2198/fabric-manufacturing-dp700.git
git push -u origin main
```

If `origin` already exists, inspect it with `git remote -v` before changing it. Do not force-push over existing work.

## Verify after publishing
- Confirm the repository loads in a private/incognito browser window.
- Confirm README and all eight lab guides render correctly.
- Open several dataset files and verify they download without authentication.
- Check that no credentials, tenant identifiers, production data or private client material were committed.
- Add a release/tag only after labs have been validated in the target Skillable environment.

Suggested description: `Hands-on Microsoft Fabric DP-700 manufacturing and supply-chain data engineering labs`.

Suggested topics: `microsoft-fabric`, `dp-700`, `data-engineering`, `manufacturing`, `supply-chain`, `databricks-migration`.
