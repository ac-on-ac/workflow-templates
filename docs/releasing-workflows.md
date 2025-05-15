# Releasing Individual Workflows

This document describes the process for releasing and versioning individual workflows in the workflow-templates repository. Versioning workflows allows consumers to reference specific, stable versions of each workflow in their CI/CD pipelines.

## The Versioning System

Workflows in this repository follow semantic versioning principles:

- **Major version**: Incremented for breaking changes that require consumers to modify their implementation
- **Minor version**: Incremented for backward-compatible feature additions
- **Patch version**: Incremented for backward-compatible bug fixes

Each workflow is versioned independently with a tag in the format:

```
<workflow-base-name>-v<major>.<minor>.<patch>
```

For example:
- `terraform-test-v1.0.0`
- `terraform-docs-v2.3.1`

## Release Process

The repository includes an automated workflow called "Release Single Workflow" that handles the versioning and release process.

### Prerequisites

Before releasing a workflow, ensure that:

1. The workflow has been properly tested
2. Any major changes are clearly documented
3. An example file exists in the `.github/examples/` directory (optional but recommended)

### Steps to Release a Workflow

1. **Navigate to the Actions tab** of the workflow-templates repository
2. **Select the "Release Single Workflow"** workflow from the list
3. **Click "Run workflow"**
4. Fill in the required information:
   - **Workflow file name**: The exact filename of the workflow to release (e.g., `reusable-terraform-test.yml`)
   - **Version type**: Choose from `patch`, `minor`, or `major` based on the nature of the changes
5. **Click "Run workflow"** to start the process

### What the Release Process Does

The release process automatically:

1. Verifies that the specified workflow file exists
2. Checks for an example file in `.github/examples/` with the same name
3. Calculates the new version number based on:
   - The latest existing tag for the workflow (if any)
   - The selected version type (patch, minor, or major)
4. Creates a Git tag using the format `<workflow-base-name>-v<version>`
5. Creates or updates a GitHub Release with:
   - The same name as the tag
   - Release notes that include the example workflow call (if available)

### Initial Release

For the first release of a workflow, the version will be set to `v1.0.0` regardless of the selected version type.

## Using Versioned Workflows

Once a workflow has been released with a version tag, consumers can reference that specific version in their GitHub Actions workflows.

### Referencing the Latest Version

```yaml
jobs:
  my-job:
    uses: your-org/workflow-templates/.github/workflows/reusable-terraform-test.yml@main
```

### Referencing a Specific Version

```yaml
jobs:
  my-job:
    uses: your-org/workflow-templates/.github/workflows/reusable-terraform-test.yml@terraform-test-v1.2.3
```

## Example Files

It is highly recommended to include an example file showing how to call your reusable workflow. This makes it easier for users to understand the required inputs and how to use the workflow.

Create your example file in the `.github/examples/` directory with the same name as your workflow file. For example:

- Workflow: `.github/workflows/reusable-terraform-test.yml`
- Example: `.github/examples/reusable-terraform-test.yml`

The example file will be automatically included in the GitHub Release notes.

## Best Practices

1. **Make semantic version choices thoughtfully**:
   - Use `patch` for bug fixes and minor improvements
   - Use `minor` for new features that don't break existing usage
   - Use `major` for breaking changes

2. **Document your changes**:
   - Keep examples up to date
   - Update documentation when workflow behavior changes

3. **Test before releasing**:
   - Ensure the workflow functions as expected before creating a release
   - Consider testing with consumers who rely on the workflow

4. **Communicate breaking changes**:
   - When releasing a major version, clearly document the breaking changes
   - Consider maintaining older versions with critical bug fixes

---

[Return to the main repository documentation](../README.md)
