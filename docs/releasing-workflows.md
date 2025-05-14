# Releasing Individual Workflow Files

This document explains how to create releases for individual GitHub workflow files in this repository.

## Overview

While GitHub releases are normally created at the repository level, this repository includes a special workflow that allows you to:

1. Create version tags for specific workflow files (e.g., `terraform-test-v1.0.0`) 
2. Generate releases that include only the specific workflow file as an asset
3. Document version history for each workflow independently

This approach allows consumers of your workflows to:
- Reference specific versions of individual workflows
- Get clear release notes specific to each workflow
- Track the evolution of each workflow independently

## Release Process

To create a release for a specific workflow file:

1. Navigate to the "Actions" tab in your repository
2. Select the "Release Single Workflow File" workflow
3. Click "Run workflow"
4. Fill in the requested inputs:
   - **Workflow file**: The name of the workflow file to release (e.g., `reusable-terraform-test.yml`)
   - **Version bump**: Choose `patch` for bug fixes, `minor` for backward-compatible additions, or `major` for breaking changes
   - **Release notes**: Document what's changed in this version (required) - a template is provided
5. Click "Run workflow" to start the release process

The workflow will:
1. Verify the workflow file exists
2. Find any previous tags for this workflow
3. Create a new tag with the pattern `{name}-v{major}.{minor}.{patch}` (omitting any "reusable-" prefix)
4. Create a GitHub release with the workflow file attached as an asset
5. Add usage instructions to the release notes

## Referencing Specific Workflow Versions

Once a workflow file has been released, other repositories can reference it by using the tag:

```yaml
jobs:
  your_job:
    uses: ac-on-ac/workflow-templates/.github/workflows/reusable-terraform-test.yml@terraform-test-v1.0.0
    with:
      # your inputs here
    secrets:
      # your secrets here
```

This ensures that even as the workflow file evolves, repositories using a specific version will continue to work correctly.

## Best Practices

1. **Release notes**: Clearly document what changed in each version. The workflow provides a template with sections for:
   ```markdown
   # Changes Made
   ## Capabilities Added
   - 
   ## Bugs Fixed
   - 
   ```
2. **Semantic versioning**: Follow semantic versioning principles:
   - Patch (0.0.X): Bug fixes, simple enhancements
   - Minor (0.X.0): New features, backward compatible
   - Major (X.0.0): Breaking changes, significant rewrites
3. **Testing**: Always test workflow changes before releasing
4. **Announce major changes**: For major version bumps, consider alerting users of the workflow

## Example Release Cycle

Here's an example release cycle for the `reusable-terraform-test.yml` workflow:

1. Initial release: `terraform-test-v0.1.0`
2. Fix a bug: `terraform-test-v0.1.1`
3. Add new features: `terraform-test-v0.2.0`
4. Breaking change: `terraform-test-v1.0.0`
