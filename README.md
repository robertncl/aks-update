# AKS Upgrade Automation with GitHub Actions

This repository provides a set of GitHub Actions workflows to automate the process of upgrading Azure Kubernetes Service (AKS) clusters. It includes features for single-cluster upgrades with issue tracking, batch upgrades, and automated status monitoring.

## Table of Contents

- [Overview](#overview)
- [Workflows](#workflows)
  - [1. Update Single AKS Cluster](#1-update-single-aks-cluster)
  - [2. Check AKS Upgrade Status from Issues](#2-check-aks-upgrade-status-from-issues)
  - [3. Update AKS Clusters (Batch)](#3-update-aks-clusters-batch)
- [Setup and Configuration](#setup-and-configuration)
  - [1. Azure Service Principal](#1-azure-service-principal)
  - [2. GitHub Secrets](#2-github-secrets)
  - [3. GitHub Actions Permissions](#3-github-actions-permissions)
- [Usage](#usage)
  - [Single Cluster Upgrade with Tracking](#single-cluster-upgrade-with-tracking)
  - [Batch Cluster Upgrades](#batch-cluster-upgrades)
- [How It Works: The Issue Tracking System](#how-it-works-the-issue-tracking-system)
- [Customization](#customization)

## Overview

Managing Kubernetes upgrades across multiple clusters can be a complex and time-consuming task. These workflows aim to simplify this process by:

- **Automating Upgrades**: Trigger AKS upgrades automatically or manually.
- **Providing Visibility**: Use GitHub Issues to track the progress of individual upgrades.
- **Enabling Batch Operations**: Update multiple clusters at once for streamlined maintenance.
- **Monitoring and Reporting**: Periodically check the status of ongoing upgrades and report back with comments on the tracking issue.

## Workflows

This repository contains three core workflows, located in the `.github/workflows/` directory.

### 1. `Update Single AKS Cluster`

- **File**: `update-single-aks.yml`
- **Purpose**: Upgrades a single AKS cluster and creates a GitHub Issue to track its progress.
- **Trigger**: Manual (`workflow_dispatch`).
- **Inputs**:
  - `resource_group`: The Azure Resource Group of the AKS cluster.
  - `cluster_name`: The name of the AKS cluster.
  - `aks_version` (Optional): The target Kubernetes version. If left blank, it will automatically use the latest available stable (non-preview) version.

### 2. `Check AKS Upgrade Status from Issues`

- **File**: `record-aks-upgrade.yml`
- **Purpose**: Monitors the status of upgrades initiated by the "Update Single AKS Cluster" workflow.
- **Trigger**: Scheduled (`cron` - runs hourly by default) and Manual (`workflow_dispatch`).
- **Process**:
  1. Finds all open GitHub Issues with the `äks-upgrade` label.
  2. Parses the issue body to get the cluster's resource group, name, and target version.
  3. Uses the Azure CLI to check the current `provisioningState` and `kubernetesVersion` of the cluster.
  4. Adds a comment to the issue with the latest status.
  5. If the upgrade is successful and the version matches the target, it closes the issue.
  6. If the upgrade has failed, it posts an error comment.

### 3. `Update AKS Clusters (Batch)`

- **File**: `update-aks.yml`
- **Purpose**: Upgrades multiple AKS clusters in a batch. This workflow does **not** use the issue tracking system.
- **Trigger**: Scheduled (`cron` - runs weekly by default) and Manual (`workflow_dispatch`).
- **Configuration**:
  - The list of clusters to be updated is configured via the `CLUSTERS` environment variable within the workflow file.
  - The format is a comma-separated string of `resourceGroup:clusterName`.
  - The `AKS_VERSION` environment variable can be set to a specific version, or left empty to upgrade each cluster to its own latest stable version.

## Setup and Configuration

Before you can use these workflows, you need to configure Azure credentials and GitHub settings.

### 1. Azure Service Principal

You need an Azure Service Principal with sufficient permissions to manage AKS clusters. The `Contributor` role on the subscription or resource group containing the clusters is typically sufficient.

Create a Service Principal using the Azure CLI:

```bash
az ad sp create-for-rbac --name "GitHubActions-AKS-Upgrade" --role Contributor --scopes /subscriptions/{your-subscription-id}
```

This command will output a JSON object containing the `clientId`, `clientSecret`, `subscriptionId`, and `tenantId`. You will need these for the next step.

### 2. GitHub Secrets

Add the following secrets to your GitHub repository (`Settings` > `Secrets and variables` > `Actions`):

- `AZURE_CLIENT_ID`: The `clientId` from the Service Principal.
- `AZURE_CLIENT_SECRET`: The `clientSecret` from the Service Principal.
- `AZURE_SUBSCRIPTION_ID`: Your Azure `subscriptionId`.
- `AZURE_TENANT_ID`: The `tenantId` from the Service Principal.

### 3. GitHub Actions Permissions

The workflows require permission to create and write to issues. Ensure that your repository's Actions settings (`Settings` > `Actions` > `General`) allow "Read and write permissions" for the `GITHUB_TOKEN`. This is also explicitly set in the workflow files, but it's good practice to have the repository setting configured correctly.

## Usage

### Single Cluster Upgrade with Tracking

1. Go to the **Actions** tab in your GitHub repository.
2. Select the **Update Single AKS Cluster** workflow from the list.
3. Click **Run workflow**.
4. Fill in the `resource_group`, `cluster_name`, and optionally the `aks_version`.
5. Click **Run workflow**.

A new GitHub Issue will be created to track the upgrade. The `Check AKS Upgrade Status from Issues` workflow will then periodically post updates to this issue until it is closed.

### Batch Cluster Upgrades

1. **Edit the workflow file**: Open `.github/workflows/update-aks.yml`.
2. **Configure clusters**: Modify the `CLUSTERS` environment variable to list the clusters you want to upgrade.
   ```yaml
   env:
     CLUSTERS: "my-rg-1:cluster-a,my-rg-2:cluster-b"
   ```
3. **Set version (optional)**: If you want to upgrade all clusters to a specific version, set the `AKS_VERSION` variable.
   ```yaml
   env:
     AKS_VERSION: "1.23.8"
   ```
4. **Run the workflow**: You can either trigger the **Update AKS Clusters** workflow manually from the Actions tab or wait for its scheduled run.

## How It Works: The Issue Tracking System

The issue tracking system is a key feature of this repository. Here’s a breakdown of the process:

1.  **Issue Creation**: When the `Update Single AKS Cluster` workflow runs, it creates a new issue with the label `äks-upgrade`. The body of this issue is populated with the cluster details and, importantly, a hidden HTML comment containing metadata:
    ```html
    <!-- metadata
    resource_group: my-rg-1
    cluster_name: cluster-a
    target_version: 1.22.11
    -->
    ```
2.  **Status Check**: The `Check AKS Upgrade Status from Issues` workflow runs on a schedule. It queries for all open issues with the `äks-upgrade` label.
3.  **Metadata Parsing**: For each issue found, the workflow parses the metadata from the HTML comment in the issue body to identify the cluster to check.
4.  **Commenting and Closing**: The workflow comments on the issue with the current status of the upgrade. If the upgrade is complete (`Succeeded` state and matching target version), it closes the issue.

This system provides a clear, auditable trail for every tracked upgrade.

## Customization

- **Schedules**: You can change the `cron` schedule in the `update-aks.yml` and `record-aks-upgrade.yml` files to fit your maintenance windows.
- **Labels**: If you want to use a different label for tracking, update it in both `update-single-aks.yml` and `record-aks-upgrade.yml`.
- **Timeouts**: The `update-single-aks.yml` workflow has a verification step with a timeout. You can adjust the `MAX_ATTEMPTS` and `INTERVAL` variables in this step if your upgrades typically take longer to start.
