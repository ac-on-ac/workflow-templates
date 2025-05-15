# Reusable Workflow Templates

This repository is designed to host reusable workflow templates for Terraform testing, documentation, and deployment.

## Available Workflows

- **[reusable-terraform-test.yml](docs/reusable-terraform-test.md)**: Runs Terraform tests for a module
- **[reusable-terraform-docs.yml](docs/reusable-terraform-docs.md)**: Generates documentation for Terraform modules
- **[reusable-release-and-tag.yml](docs/reusable-release-and-tag.md)**: Creates releases and tags for repositories

## Versioning Individual Workflows

This repository supports creating releases for individual workflow files, allowing consumers to reference specific versions of each workflow. This ensures stability in your CI/CD pipelines even as workflows evolve.

See [Releasing Individual Workflows](docs/releasing-workflows.md) for details on how to create and use versioned workflows.

### Example Workflow Call Files

It is highly recommended to include an example of how to call your reusable workflow. Create a YAML file in the `.github/examples/` directory with the same name as your workflow file (e.g., if your workflow is `.github/workflows/reusable-terraform-plan.yml`, the example file should be `.github/examples/reusable-terraform-plan.yml`).

This example file will be automatically included in the GitHub Release notes, making it easier for users to understand how to use your workflow.

## Usage Examples

To use these workflows in your repository:

```yaml
jobs:
  terraform-testing:
    uses: ac-on-ac/workflow-templates/.github/workflows/reusable-terraform-test.yml@main
    with:
      test_directory: tests
    secrets:
      azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
```

For a specific version:

```yaml
jobs:
  terraform-testing:
    uses: ac-on-ac/workflow-templates/.github/workflows/reusable-terraform-test.yml@terraform-test-v1.0.0
    with:
      test_directory: tests
    secrets:
      azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
```
