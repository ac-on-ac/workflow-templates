# Reusable Workflow Templates

This repository is designed to host reusable workflow templates for Terraform testing, documentation, and deployment.

## Available Workflows

- **[reusable-terraform-test.yml](docs/reusable-terraform-test.md)**: Runs Terraform tests for a module
- **[reusable-terraform-docs.yml](docs/reusable-terraform-docs.md)**: Generates documentation for Terraform modules
- **[reusable-release-and-tag.yml](docs/reusable-release-and-tag.md)**: Creates releases and tags for repositories

## Versioning Individual Workflows

This repository supports creating releases for individual workflow files, allowing consumers to reference specific versions of each workflow. This ensures stability in your CI/CD pipelines even as workflows evolve.

The release process now supports two methods for providing release notes:
1. **File-based release notes** (recommended): Create and update files in `.github/release-notes/` directory
2. **Inline release notes**: Directly in the workflow dispatch form

See [Releasing Individual Workflows](docs/releasing-workflows-improved.md) for details on how to create and use versioned workflows.

See [Releasing the Terraform Test Workflow](docs/releasing-terraform-test-workflow-improved.md) for a specific example.

## Releasing Workflows

To release a new version of a reusable workflow, use the "Release Single Workflow" GitHub Actions workflow.

1.  **Navigate to the Actions tab** of the `workflow-templates` repository.
2.  **Select the "Release Single Workflow"** workflow from the list.
3.  **Click "Run workflow"**.
4.  **Enter the filename of the workflow** you want to release (e.g., `reusable-terraform-plan.yml`) in the "Workflow file name" input.
5.  **Choose the version type** (patch, minor, or major) for the release.
6.  **Click "Run workflow"**.

This will trigger the workflow to:
    - Determine the current version of the specified workflow.
    - Calculate the next semantic version based on your input.
    - Create a new Git tag in the format `<workflow-base-name>-v<new-version>`.
    - Push the new tag to the repository.
    - Create (or update if it already exists) a GitHub Release associated with the new tag.

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
