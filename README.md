# AKS Update GitHub Actions & Issue Tracking

This repository contains GitHub Actions workflows to automate and track the upgrade of Azure Kubernetes Service (AKS) clusters.

## `Update Single AKS Cluster` Workflow Setup & Process

1. **Azure Credentials:**
   - Add the following secrets to your GitHub repository:
     - `AZURE_CLIENT_ID`
     - `AZURE_TENANT_ID`
     - `AZURE_SUBSCRIPTION_ID`
     - `AZURE_CLIENT_SECRET`

2. **Permissions:**
   - Ensure the `GITHUB_TOKEN` has `issues: write` permissions. This can be set in repository settings or within the workflow file. The provided `update-single-aks.yml` workflow includes this permission.

3. **Run Workflow:**
   - Trigger the `Update Single AKS Cluster` workflow manually via the GitHub Actions tab.
   - Provide the `resource_group`, `cluster_name`, and optionally, a specific `aks_version`. If `aks_version` is empty, it will attempt to upgrade to the latest available non-preview version.

4. **Issue Tracking:**
   - Upon initiation, the workflow creates a GitHub Issue with the label `äks-upgrade`.
   - This issue will contain details of the upgrade (Resource Group, Cluster Name, Target Version) and a link to the workflow run.
   - The `Check AKS Upgrade Status from Issues` workflow runs hourly (by default) to:
     - Find all open issues with the `äks-upgrade` label.
     - Check the current status and Kubernetes version of the corresponding AKS cluster using Azure CLI.
     - Add a comment to the issue with the latest status.
     - Close the issue if the upgrade to the target version is confirmed as "Succeeded".
     - Comment on the issue if the upgrade has "Failed" or is still in progress.

## Other Workflows

- **`Update AKS Clusters` (`update-aks.yml`):** This workflow can be used for batch updates based on a schedule or manual trigger. It does not currently integrate with the GitHub Issue tracking system used by `Update Single AKS Cluster`.
- **`Check AKS Upgrade Status from Issues` (`record-aks-upgrade.yml`):** This workflow periodically checks and updates the status of upgrades tracked in GitHub Issues.

---
See the respective workflow files in `.github/workflows/` for more details.
