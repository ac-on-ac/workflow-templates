# Reusable Terraform Static Analysis Workflow

This document provides an overview of the reusable GitHub Actions workflow for running static analysis against Terraform codebases.

## Purpose

The `reusable-terraform-static-analysis.yml` workflow automates the process of running static analysis tools against Terraform code in the calling repository. It runs the following tools in parallel:

- **[tflint](https://github.com/terraform-linters/tflint)**: A Terraform linter that detects possible errors, enforces best practices, and supports provider-specific rules via plugins
- **[checkov](https://www.checkov.io/)**: A static analysis security scanner that identifies misconfigurations and security issues in Terraform code

This workflow is designed to:

1. Catch linting errors and security misconfigurations early in the development cycle
2. Enforce consistent code quality standards across Terraform repositories
3. Provide fast, parallel feedback by running both tools simultaneously
4. Leverage existing repository configuration (`.tflint.hcl`) for tflint plugin and rule management

## Prerequisites

The calling repository must contain a `.tflint.hcl` configuration file. This file is used by tflint to initialise plugins and configure linting rules. It should be placed in the directory that static analysis is run against (by default, the repository root).

Example `.tflint.hcl`:

```hcl
plugin "aws" {
  enabled = true
  version = "0.38.0"
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}

rule "terraform_naming_convention" {
  enabled = true
}

rule "terraform_documented_variables" {
  enabled = true
}
```

## Workflow Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `working_directory` | No | `.` (repository root) | Directory to run static analysis against |

## Permissions

The workflow requires the following permissions:

- `contents: read`: Allows reading the repository content for analysis

No write permissions are required, as this workflow only reads and analyses code.

## Workflow Process

The workflow consists of two independent jobs that run in parallel:

### 1. tflint

This job:
1. Checks out the calling repository
2. Installs tflint
3. Initialises tflint plugins as defined in the repository's `.tflint.hcl` file (uses `GITHUB_TOKEN` to avoid GitHub API rate limits when downloading plugins)
4. Runs tflint recursively across the working directory

### 2. checkov

This job:
1. Checks out the calling repository
2. Installs checkov via pip
3. Runs checkov against the working directory, scoped to the Terraform framework

## Usage Example

To use this workflow in your GitHub repository, create a workflow file like this:

```yaml
name: Terraform Static Analysis

on:
  pull_request:
    branches: [ main ]
    paths:
      - '**.tf'
      - '.tflint.hcl'

jobs:
  static-analysis:
    uses: ac-on-ac/workflow-templates/.github/workflows/reusable-terraform-static-analysis.yml@static-analysis-v1.0.0
    with:
      working_directory: "."
```

## Configuration

### tflint

tflint behaviour is controlled entirely by the `.tflint.hcl` file in the calling repository. This includes:

- **Plugins**: Provider-specific rulesets (e.g. `tflint-ruleset-aws`, `tflint-ruleset-azurerm`)
- **Rules**: Which rules to enable or disable and their configuration
- **Configuration options**: Such as `call_module_type` for module inspection

Refer to the [tflint documentation](https://github.com/terraform-linters/tflint/blob/master/docs/user-guide/config.md) for a full list of configuration options.

### checkov

checkov runs with the `--framework terraform` flag, limiting its checks to Terraform-specific policies. It will scan all `.tf` files within the working directory recursively and report on any policy violations.

To suppress specific checkov checks inline, use a `checkov` skip comment in your Terraform code:

```hcl
resource "aws_s3_bucket" "example" {
  bucket = "example"
  #checkov:skip=CKV_AWS_18:Access logging not required for this bucket
}
```

Refer to the [checkov documentation](https://www.checkov.io/2.Basics/Suppressing%20and%20Skipping%20Policies.html) for further suppression options.

## Error Handling

The workflow will fail if:

- tflint reports any linting errors based on the rules configured in `.tflint.hcl`
- checkov detects any Terraform security misconfigurations or policy violations
- The `.tflint.hcl` file is missing or malformed, causing tflint initialisation to fail

Both jobs run independently, so a failure in one does not prevent the other from completing. This ensures you always receive feedback from both tools in a single workflow run.

---

[Return to the main repository documentation](../README.md)