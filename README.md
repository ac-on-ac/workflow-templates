# Reusable Workflow Templates

This repository is designed to host reusable workflow templates for Terraform testing, documentation, and deployment.

## Available Workflows

- **reusable-terraform-test.yml**: Runs Terraform tests for a module
- **reusable-terraform-docs.yml**: Generates documentation for Terraform modules
- **reusable-release-and-tag.yml**: Creates releases and tags for repositories
- **release-single-workflow.yml**: Creates releases for individual workflow files

## Versioning Individual Workflows

This repository supports creating releases for individual workflow files, allowing consumers to reference specific versions of each workflow. This ensures stability in your CI/CD pipelines even as workflows evolve.

The release process includes a standardized release notes format with sections for:
```markdown
# Changes Made
## Capabilities Added
- 
## Bugs Fixed
- 
```

See [Releasing Individual Workflows](docs/releasing-workflows.md) for details on how to create and use versioned workflows.

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
