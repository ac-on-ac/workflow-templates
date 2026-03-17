# Reusable Release and Tag Workflow

This document provides an overview of the reusable GitHub Actions workflow for automatically creating semantic versioned releases and tags based on pull request information.

## Purpose

The `reusable-release-and-tag.yml` workflow automates the process of creating new version tags and GitHub releases when pull requests are merged. It follows semantic versioning principles based on the PR title prefix, allowing for consistent and predictable versioning across your repositories. It supports both stable releases and release candidates.

This workflow is designed to:

1. Enforce semantic versioning practices
2. Automate the release and tagging process
3. Create consistent release notes from PR descriptions
4. Ensure version numbers are incremented appropriately based on change type
5. Support pre-release (release candidate) workflows before committing to a stable version

## Workflow Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `pr_number` | Yes | The pull request number that triggered the workflow |
| `pr_merged` | Yes | Boolean indicating whether the PR was merged or not |

## Workflow Outputs

| Output | Description |
|--------|-------------|
| `new_tag` | The newly created version tag (e.g. `v1.2.3` or `v1.3.0-rc.1`) |

## Permissions

The workflow requires the following permissions:
- `pull-requests: read`: Allows reading PR information
- `contents: write`: Allows creating tags and releases

## Workflow Process

The workflow consists of a single job that:

1. Checks if the PR was merged (exits if not)
2. Fetches all repository history and tags
3. Gets the latest existing **stable** tag, ignoring any release candidate (`-rc.*`) tags
4. Extracts the semantic version numbers from the latest stable tag
5. Retrieves the pull request title and description
6. Validates the PR title format (must begin with one of the accepted prefixes — see [PR Title Requirements](#pr-title-requirements))
7. Determines the new version based on:
   - The latest stable tag (or initialises to `v0.0.1`, `v0.1.0`, or `v1.0.0` if no stable tags exist)
   - The type of change indicated in the PR title prefix
   - For RC prefixes: finds the highest existing RC tag for the calculated base version and increments the counter, or starts at `rc.1`
8. Creates a new Git tag and pushes it to the repository
9. Creates or updates a GitHub release with the PR description as release notes, marked as a pre-release if the tag contains `-rc.`

## PR Title Requirements

For this workflow to function correctly, pull request titles must follow this format:

### Stable releases

| Prefix | Effect |
|--------|--------|
| `patch:` | Increments patch component — `v1.2.3` → `v1.2.4` |
| `minor:` | Increments minor component, resets patch — `v1.2.3` → `v1.3.0` |
| `major:` | Increments major component, resets minor and patch — `v1.2.3` → `v2.0.0` |

### Release candidates

| Prefix | Effect |
|--------|--------|
| `rc-patch:` | Next patch RC — `v1.2.3` → `v1.2.4-rc.1`, then `v1.2.4-rc.2`, ... |
| `rc-minor:` | Next minor RC — `v1.2.3` → `v1.3.0-rc.1`, then `v1.3.0-rc.2`, ... |
| `rc-major:` | Next major RC — `v1.2.3` → `v2.0.0-rc.1`, then `v2.0.0-rc.2`, ... |

Examples:
- `patch: Fix a bug in subnet creation`
- `minor: Add support for NAT gateway`
- `major: Complete redesign of network architecture`
- `rc-patch: Fix a bug in subnet creation (release candidate)`
- `rc-minor: Add support for NAT gateway (release candidate)`
- `rc-major: Complete redesign of network architecture (release candidate)`

If the PR title doesn't follow this format, the workflow will fail with an error message.

## Version Numbering

The workflow follows semantic versioning (MAJOR.MINOR.PATCH):
1. MAJOR version increments for incompatible API changes
2. MINOR version increments for backwards-compatible functionality additions
3. PATCH version increments for backwards-compatible bug fixes

If no stable tags exist in the repository:
- The first `patch:` or `rc-patch:` PR creates `v0.0.1` or `v0.0.1-rc.1`
- The first `minor:` or `rc-minor:` PR creates `v0.1.0` or `v0.1.0-rc.1`
- The first `major:` or `rc-major:` PR creates `v1.0.0` or `v1.0.0-rc.1`

### RC counter behaviour

The RC counter increments automatically based on existing tags. If `v1.3.0-rc.2` already exists, the next `rc-minor:` PR targeting the same base version produces `v1.3.0-rc.3`. RC tags never affect the stable version baseline — stable releases always bump from the latest stable tag.

### GitHub release pre-release flag

Releases created from RC tags are automatically marked as **pre-releases** on GitHub (`prerelease: true`). Stable releases are marked as full releases (`prerelease: false`).

## Example Calls

To reuse this workflow in your GitHub repository, include it as a job in your workflow with one of the following patterns:

### Referencing the Main Branch

```yaml
name: Create Release

on:
  pull_request:
    types: [closed]
    branches: [main]

jobs:
  release:
    if: github.event.pull_request.merged == true
    uses: ac-on-ac/workflow-templates/.github/workflows/reusable-release-and-tag.yml@main
    with:
      pr_number: ${{ github.event.pull_request.number }}
      pr_merged: ${{ github.event.pull_request.merged }}
```

### Referencing a Specific Release Version

```yaml
name: Example Using Release and Tag Workflow

on:
  pull_request:
    types: [closed]
    branches: [main]

jobs:
  call_reusable_workflow:
    uses: ac-on-ac/workflow-templates/.github/workflows/reusable-release-and-tag.yml@release-and-tag-v2.0.0 # Replace with desired version
    if: github.event.pull_request.merged == true
    with:
      pr_number: ${{ github.event.pull_request.number }}
      pr_merged: ${{ github.event.pull_request.merged }}
```

## Best Practices

1. Include detailed descriptions in your pull requests, as these become your release notes
2. Use the appropriate prefix in your PR title to indicate the type of change
3. Use `rc-*` prefixes to publish pre-releases for validation before promoting to a stable version
4. Consider using the workflow's `new_tag` output to reference the newly created tag in subsequent steps

---

[Return to the main repository documentation](../README.md)
