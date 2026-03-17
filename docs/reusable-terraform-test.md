# Reusable Terraform Test Workflow

This document provides an overview of the reusable GitHub Actions workflow for testing Terraform modules.

## Purpose

The `reusable-terraform-test.yml` workflow is designed to automatically discover and execute Terraform tests from a specified directory. It supports three categories of tests:

- **Validation Tests** (`validation-*.tftest.hcl`): Verify the module's configuration and input validations
- **Unit Tests** (`unit-*.tftest.hcl`): Verify individual components or functions within the module
- **Integration Tests** (`integration-*.tftest.hcl`): Verify the module's integration with other resources or services

This workflow enables consistent testing across Terraform modules and can be reused by multiple repositories, reducing duplication of test configuration. All secrets are optional — pass only the credentials your module requires.

## Workflow Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `test_directory` | Yes | The directory containing the Terraform test files to run |

## Workflow Secrets

All secrets are optional. Pass only the credentials your module needs.

| Secret | Required | Description |
|--------|----------|-------------|
| `azure_credentials` | No | Azure service principal credentials (JSON) for authenticating with Azure. Required for modules that deploy Azure resources and for Azure Databricks authentication. When provided, the Azure Login step runs before `terraform init`. |
| `terraform_token` | No | API token for Terraform Cloud or Terraform Enterprise (sets `TF_TOKEN_app_terraform_io`). Applied at the job level — available to `terraform init` and all `terraform test` invocations. Required for modules that interact with TFC/TFE. |
| `databricks_host` | No | Databricks workspace URL (e.g. `https://adb-<id>.<region>.azuredatabricks.net`). Sets `DATABRICKS_HOST`. Must be paired with `azure_credentials` and `databricks_azure_workspace_resource_id` for Azure Databricks authentication. |
| `databricks_azure_workspace_resource_id` | No | Full Azure resource ID of the Databricks workspace (e.g. `/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Databricks/workspaces/<name>`). Sets `DATABRICKS_AZURE_RESOURCE_ID`. Required alongside `databricks_host` for Azure Databricks authentication. |

### Credential combinations by module type

| Module type | Secrets to pass |
|-------------|-----------------|
| Azure only | `azure_credentials` |
| TFC/TFE only | `terraform_token` |
| Azure Databricks | `azure_credentials` + `databricks_host` + `databricks_azure_workspace_resource_id` |
| Mock/validation only | _(none)_ |
| Hybrid | All that apply |

## Permissions

The workflow requires the following permissions:
- `contents: write`: Allows writing to the repository content
- `pull-requests: write`: Allows writing to pull requests

## Workflow Process

The workflow consists of two jobs:

### 1. find-test-files

This job:
1. Checks out the repository code
2. Discovers test files in the specified directory matching the naming patterns:
   - `validation-*.tftest.hcl`
   - `unit-*.tftest.hcl`
   - `integration-*.tftest.hcl`
3. Categorizes the files into validation, unit, and integration tests
4. Verifies that at least one test file exists
5. Outputs the categorized test file lists for use by the next job

### 2. terraform-test

This job:
1. Checks out the repository code
2. Installs Terraform
3. **Conditionally** authenticates with Azure using `azure/login` — this step is skipped when `azure_credentials` is not provided
4. Initialises Terraform (`TF_TOKEN_app_terraform_io` is available at the job level, so private module sources in TFC/TFE are accessible)
5. Sequentially runs each test category in the following order, with `TF_TOKEN_app_terraform_io` in scope throughout:
   - Validation tests
   - Unit tests
   - Integration tests
6. Each test is executed with appropriate filtering to run only the specific test file

### Databricks authentication

When `databricks_host` and `databricks_azure_workspace_resource_id` are provided, the workflow sets `DATABRICKS_HOST` and `DATABRICKS_AZURE_RESOURCE_ID` as job-level environment variables. The Databricks Terraform provider resolves these automatically. For Azure-hosted Databricks workspaces, the Azure CLI session established by the Azure Login step is used for authentication — no separate Databricks login step is required.

## Test File Requirements

Test files must follow this naming convention:
- `validation-*.tftest.hcl`: For validation tests
- `unit-*.tftest.hcl`: For unit tests
- `integration-*.tftest.hcl`: For integration tests

All test files must be located in the directory specified by the `test_directory` input.

## Example Calls

To reuse this workflow in your GitHub repository, you can reference it in your workflow file using one of the following patterns:

### Referencing the Main Branch

```yaml
name: Run Terraform Tests

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    uses: ac-on-ac/workflow-templates/.github/workflows/reusable-terraform-test.yml@main
    with:
      test_directory: 'tests'
    secrets:
      azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}                                 # Optional
      terraform_token: ${{ secrets.TFE_TOKEN }}                                           # Optional
      databricks_host: ${{ secrets.DATABRICKS_HOST }}                                     # Optional
      databricks_azure_workspace_resource_id: ${{ secrets.DATABRICKS_AZURE_RESOURCE_ID }} # Optional
```

### Referencing a Specific Release Version

```yaml
name: Example Call for Reusable Terraform Test Workflow

on:
  pull_request:
    types: [opened, synchronize, reopened]
    branches: [main]

jobs:
  call-terraform-test:
    uses: ac-on-ac/workflow-templates/.github/workflows/reusable-terraform-test.yml@terraform-test-v2.0.0 # Replace with desired version
    with:
      test_directory: tests
    secrets:
      azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}                                 # Optional — Azure modules and Azure Databricks modules
      terraform_token: ${{ secrets.TFE_TOKEN }}                                           # Optional — TFC/TFE modules only
      databricks_host: ${{ secrets.DATABRICKS_HOST }}                                     # Optional — Databricks modules only
      databricks_azure_workspace_resource_id: ${{ secrets.DATABRICKS_AZURE_RESOURCE_ID }} # Optional — Azure Databricks modules only
```

## Error Handling

The workflow will fail if:
- No test files are found matching any of the expected naming patterns
- Any of the tests fail during execution

This ensures that all tests must pass for the workflow to succeed.

---

[Return to the main repository documentation](../README.md)
