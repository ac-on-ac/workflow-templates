# Reusable Terraform Docs Workflow

This document provides an overview of the reusable GitHub Actions workflow for automatically generating documentation for Terraform modules.

## Purpose

The `reusable-terraform-docs.yml` workflow automates the process of generating and updating documentation for Terraform modules using the [terraform-docs](https://terraform-docs.io/) tool. It ensures your README files are always up-to-date and consistent with your Terraform code, including inputs, outputs, providers, and requirements.

This workflow is designed to:

1. Enforce consistent and accurate module documentation
2. Automate the documentation update process
3. Reduce manual documentation effort
4. Integrate documentation generation into your CI/CD pipeline

## Workflow Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `terraform_docs_config_file` | No | `.terraform-docs.yml` | Path to the terraform-docs configuration file |
| `working_directory` | No | `.` (repository root) | Directory to generate docs in |

## Workflow Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `token` | No | GitHub token used to commit and push documentation changes. Pass a Personal Access Token (PAT) with `contents:write` permission to allow the terraform-docs commit to re-trigger downstream CI workflows on the updated PR HEAD. If omitted, falls back to `GITHUB_TOKEN`, which will **not** re-trigger workflows — required status checks will show as "Waiting for status to be reported" after the docs commit. See [CI Re-triggering](#ci-re-triggering) below. |

## Workflow Outputs

This workflow does not produce explicit outputs, but it updates your documentation files (such as `README.md`) in the repository.

## Permissions

The workflow requires the following permissions:
- `contents: write`: Allows writing to the repository content
- `pull-requests: write`: Allows writing to pull requests

## Workflow Process

The workflow consists of a single job that:

1. Checks out the repository code with full history, using the provided `token` secret (or `GITHUB_TOKEN` if not supplied)
2. Installs the terraform-docs tool (v0.20.0)
3. Generates README.md files according to the configuration
4. Removes the downloaded terraform-docs archive from the repository and runner
5. Commits and pushes the changes back to the repository
   - Handles potential merge conflicts automatically through direct push, rebasing, or creating a temporary branch and merging

## CI Re-triggering

By default, GitHub Actions does not re-trigger workflows on commits made by `github-actions[bot]` (i.e. commits pushed using `GITHUB_TOKEN`). This means that when this workflow commits regenerated documentation back to the PR branch, the new commit becomes the PR HEAD with no check results — leaving required status checks in a **"Waiting for status to be reported"** state.

To resolve this, pass a Personal Access Token (PAT) via the `token` secret. Commits pushed using a PAT are attributed to the PAT owner (a real user account) and **will** re-trigger workflows.

### Setting up the PAT

The token must belong to a **dedicated service/bot account**, not a personal account. If the account is deactivated or leaves the organisation, CI in every repo using this workflow will break. The account must have **Write access** to each repository that calls this workflow.

**Step 1 — Create the PAT on the bot account**

- **Fine-grained PAT (recommended):** GitHub → bot account Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token. Set repository access to only the repositories that use this workflow. Under Repository permissions, set **Contents: Read and write**.
- **Classic PAT:** GitHub → bot account Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token. Check the `repo` scope.

**Step 2 — Store the PAT as a repository secret**

For each repository that calls this workflow: Repository → Settings → Secrets and variables → Actions → New repository secret. Name: `TERRAFORM_DOCS_PAT`. Value: the PAT from Step 1.

**Step 3 — Pass the secret in your workflow call**

```yaml
secrets:
  token: ${{ secrets.TERRAFORM_DOCS_PAT }}
```

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

## Example Calls

To reuse this workflow in your GitHub repository, you can reference it in your workflow file using one of the following patterns:

### Referencing the Main Branch

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
    uses: ac-on-ac/workflow-templates/.github/workflows/reusable-terraform-docs.yml@main
    with:
      terraform_docs_config_file: ".terraform-docs.yml"
      working_directory: "."
    secrets:
      token: ${{ secrets.TERRAFORM_DOCS_PAT }} # Optional — omit to use GITHUB_TOKEN (checks will not re-trigger)
```

### Referencing a Specific Release Version

```yaml
name: Example Call for Reusable Terraform Docs Workflow

on:
  pull_request:
    types: [opened, synchronize, reopened]
    branches: [main]

jobs:
  call-terraform-docs:
    uses: ac-on-ac/workflow-templates/.github/workflows/reusable-terraform-docs.yml@terraform-docs-v2.0.0 # Replace with desired version
    with:
      terraform_docs_config_file: .terraform-docs.yml
      working_directory: .
    secrets:
      token: ${{ secrets.TERRAFORM_DOCS_PAT }} # Optional — omit to use GITHUB_TOKEN (checks will not re-trigger)
```

## Best Practices

1. Include a `.terraform-docs.yml` file in your Terraform module repository
2. Add a trigger to your workflow that runs when Terraform files change
3. Configure the workflow to run as part of your pull request process
4. Use comment markers in your README.md to specify where generated documentation should be injected
5. Set up the `TERRAFORM_DOCS_PAT` secret to ensure required status checks re-trigger after the docs commit

---

[Return to the main repository documentation](../README.md)
