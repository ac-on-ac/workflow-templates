# Reusable Terraform Docs Workflow

This document provides an overview of the reusable GitHub Actions workflow for automatically generating documentation for Terraform modules.

## Purpose

The `reusable-terraform-docs.yml` workflow automates the process of generating documentation for Terraform modules using the [terraform-docs](https://terraform-docs.io/) tool. It updates README files with standardized documentation based on your Terraform code, including inputs, outputs, providers, and requirements.

This workflow is designed to:

1. Keep module documentation consistent and up-to-date
2. Reduce manual documentation effort
3. Ensure documentation accuracy by deriving it directly from code
4. Generate documentation as part of your CI/CD pipeline

## Workflow Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `terraform_docs_config_file` | No | `.terraform-docs.yml` | Path to the terraform-docs configuration file |
| `working_directory` | No | `.` (repository root) | Directory to generate docs in |

## Permissions

The workflow requires the following permissions:
- `contents: write`: Allows writing to the repository content
- `pull-requests: write`: Allows writing to pull requests

## Workflow Process

The workflow consists of a single job that:

1. Checks out the repository code with full history
2. Installs the terraform-docs tool (v0.20.0)
3. Generates README.md files according to the configuration
4. Commits and pushes the changes back to the repository

The commit process includes handling potential merge conflicts automatically through various strategies:
- Direct push attempt
- Rebasing if direct push fails
- Creating a temporary branch and merging if rebasing fails

## Configuration File

This workflow expects a terraform-docs configuration file (by default named `.terraform-docs.yml`) in your repository. This file defines how documentation should be generated.

Example configuration file:

```yaml
formatter: "markdown table"

sections:
  hide:
    - providers

content: |-
  {{ .Header }}

  {{ .Requirements }}

  {{ .Inputs }}

  {{ .Outputs }}

  {{ .Resources }}

  {{ .Modules }}

output:
  file: "README.md"
  mode: inject
  template: |-
    <!-- BEGIN_TF_DOCS -->
    {{ .Content }}
    <!-- END_TF_DOCS -->
```

## Usage Example

To use this workflow in your GitHub repository, create a workflow file like this:

```yaml
name: Update Terraform Documentation

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
    paths:
      - '**.tf'
      - '.terraform-docs.yml'

jobs:
  generate-docs:
    uses: your-org/workflow-templates/.github/workflows/reusable-terraform-docs.yml@main
    with:
      terraform_docs_config_file: ".terraform-docs.yml"
      working_directory: "."
```

## Recommended Practices

1. Include a `.terraform-docs.yml` file in your Terraform module repository
2. Add a trigger to your workflow that runs when Terraform files change
3. Configure the workflow to run as part of your pull request process
4. Use comment markers in your README.md to specify where generated documentation should be injected

---

[Return to the main repository documentation](../README.md)
