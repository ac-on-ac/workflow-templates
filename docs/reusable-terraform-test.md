# Reusable Terraform Test Workflow

This document provides an overview of the reusable GitHub Actions workflow for testing Terraform modules.

## Purpose

The `reusable-terraform-test.yml` workflow is designed to automatically discover and execute Terraform tests from a specified directory. It handles three categories of tests:

- **Validation Tests** (`validation-*.tftest.hcl`): Tests that verify the module's configuration and input validations
- **Unit Tests** (`unit-*.tftest.hcl`): Tests that verify individual components or functions within the module
- **Integration Tests** (`integration-*.tftest.hcl`): Tests that verify the module's integration with other resources or services

This workflow enables consistent testing across Terraform modules and can be reused by multiple repositories, reducing duplication of test configuration.

## Workflow Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `test_directory` | Yes | The directory containing the Terraform test files to run |

## Workflow Secrets

| Secret | Required | Description |
|-------|----------|-------------|
| `azure_credentials` | Yes | Azure credentials for authenticating with Azure during tests |

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
3. Authenticates with Azure using the provided credentials
4. Initializes Terraform
5. Sequentially runs each test in the following order:
   - Validation tests
   - Unit tests
   - Integration tests
6. Each test is executed with appropriate filtering to run only the specific test file

## Test File Requirements

The workflow requires test files to follow this naming convention:
- `validation-*.tftest.hcl`: For validation tests
- `unit-*.tftest.hcl`: For unit tests
- `integration-*.tftest.hcl`: For integration tests

All test files must be located in the directory specified by the `test_directory` input.

## Usage Example

To use this workflow in your GitHub repository, create a workflow file like this:

```yaml
name: Run Terraform Tests

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    uses: your-org/workflow-templates/.github/workflows/reusable-terraform-test.yml@main
    with:
      test_directory: 'tests'
    secrets:
      azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
```

## Error Handling

The workflow will fail if:
- No test files are found matching any of the expected naming patterns
- Any of the tests fail during execution

This ensures that all tests must pass for the workflow to succeed.

---

[Return to the main repository documentation](../README.md)
