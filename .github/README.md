# AKS Update GitHub Action

This repository contains a GitHub Actions workflow to automate the upgrade of Azure Kubernetes Service (AKS) clusters to a specified or latest version.

## Setup

1. **Azure Credentials:**
   - Add the following secrets to your GitHub repository:
     - `AZURE_CLIENT_ID`
     - `AZURE_TENANT_ID`
     - `AZURE_SUBSCRIPTION_ID`
     - `AZURE_CLIENT_SECRET`

2. **Configure Clusters:**
   - Edit the `CLUSTERS` environment variable in `.github/workflows/update-aks.yml`.
   - Format: `resourceGroup1:clusterName1,resourceGroup2:clusterName2,...`

3. **AKS Version:**
   - Set `AKS_VERSION` to a specific version or leave empty to upgrade to the latest available version.

4. **Run Workflow:**
   - Trigger manually via GitHub Actions or wait for the scheduled run (default: every Sunday at 2 AM UTC).

---

See `.github/workflows/update-aks.yml` for details.
