# Reusable Release and Tag Workflow

This document provides an overview of the reusable GitHub Actions workflow for automatically creating semantic versioned releases and tags based on pull request information.

## Purpose

The `reusable-release-and-tag.yml` workflow automates the process of creating new version tags and GitHub releases when pull requests are merged. It follows semantic versioning principles based on the PR title prefix, allowing for consistent and predictable versioning across your repositories.

This workflow is designed to:

1. Enforce semantic versioning practices
2. Automate the release and tagging process
3. Create consistent release notes from PR descriptions
4. Ensure version numbers are incremented appropriately based on change type

## Workflow Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `pr_number` | Yes | The pull request number that triggered the workflow |
| `pr_merged` | Yes | Boolean indicating whether the PR was merged or not |

## Workflow Outputs

| Output | Description |
|--------|-------------|
| `new_tag` | The newly created version tag (e.g., v1.2.3) |

## Permissions

The workflow requires the following permissions:
- `pull-requests: read`: Allows reading PR information
- `contents: write`: Allows creating tags and releases

## Workflow Process

The workflow consists of a single job that:

1. Checks if the PR was merged (exits if not)
2. Fetches all repository history and tags
3. Gets the latest existing tag (if any)
4. Extracts the semantic version numbers from the latest tag
5. Retrieves the pull request title and description
6. Validates the PR title format (must begin with "patch:", "minor:", or "major:")
7. Determines the new version number based on:
   - The latest tag (or initializes to v0.0.1, v0.1.0, or v1.0.0 if no tags exist)
   - The type of change indicated in the PR title prefix
8. Creates a new Git tag and pushes it to the repository
9. Creates or updates a GitHub release with the PR description as the release notes

## PR Title Requirements

For this workflow to function correctly, pull request titles must follow this format:
- `patch:` for bug fixes and minor changes that don't add functionality
- `minor:` for backwards-compatible new features
- `major:` for breaking changes

Examples:
- `patch: Fix a bug in subnet creation`
- `minor: Add support for NAT gateway`
- `major: Complete redesign of network architecture`

If the PR title doesn't follow this format, the workflow will fail with an error message.

## Version Numbering

The workflow follows semantic versioning (MAJOR.MINOR.PATCH):
1. MAJOR version increments for incompatible API changes
2. MINOR version increments for backwards-compatible functionality additions
3. PATCH version increments for backwards-compatible bug fixes

If no tags exist in the repository:
- The first `patch:` PR creates v0.0.1
- The first `minor:` PR creates v0.1.0
- The first `major:` PR creates v1.0.0

## Usage Example

To use this workflow in your GitHub repository, create a workflow file like this:

```yaml
name: Create Release

on:
  pull_request:
    types: [closed]
    branches: [main]

jobs:
  release:
    if: github.event.pull_request.merged == true
    uses: your-org/workflow-templates/.github/workflows/reusable-release-and-tag.yml@main
    with:
      pr_number: ${{ github.event.pull_request.number }}
      pr_merged: ${{ github.event.pull_request.merged }}
```

## Best Practices

1. Include detailed descriptions in your pull requests, as these become your release notes
2. Use the appropriate prefix in your PR title to indicate the type of change
3. Consider using the released version tag in other workflows or dependencies
4. Use the workflow's outputs to reference the newly created tag in subsequent steps

---

[Return to the main repository documentation](../README.md)
