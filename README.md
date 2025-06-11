# aks-update
Repository for AKS update related code and resources

# AKS Update Workflow

This GitHub Actions workflow updates the AKS version of different AKS clusters. It is triggered manually or on a schedule.

- Requires Azure credentials stored as repository secrets: `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`, `AZURE_CLIENT_SECRET`.
- Edit the `CLUSTERS` environment variable in the workflow to specify your AKS clusters (format: `resourceGroup1:clusterName1,resourceGroup2:clusterName2,...`).
- Optionally, set `AKS_VERSION` to a specific version or leave empty to upgrade to the latest available.

---
