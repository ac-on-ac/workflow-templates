# Reusable Check PR Title Workflow

This document provides an overview of the reusable GitHub Actions workflow for validating pull request titles according to semantic versioning conventions.

## Purpose

The `reusable-check-pr-title.yml` workflow enforces that pull request titles begin with a recognised semantic versioning prefix. This ensures consistency and compatibility with workflows that automate releases and tagging based on PR titles.

This workflow is designed to:

1. Enforce semantic versioning practices for PR titles
2. Provide immediate feedback to contributors if the PR title format is incorrect
3. Prevent downstream release workflows from failing due to invalid PR titles

## Workflow Inputs

This workflow does **not** require any inputs.

## Workflow Secrets

This workflow does **not** require any secrets.

## Workflow Outputs

This workflow does **not** produce explicit outputs. It will fail with a descriptive error if the PR title does not match the required format.

## Permissions

The workflow requires the following permissions:
- `pull-requests: read`: Allows reading PR information

## Workflow Process

The workflow consists of a single job that:

1. Retrieves the pull request title using the GitHub API
2. Validates that the title starts with one of the accepted prefixes (see [PR Title Requirements](#pr-title-requirements))
3. Fails with a descriptive error message if the title does not match the required format, including examples of all valid PR title prefixes

## PR Title Requirements

Pull request titles must begin with one of the following prefixes:

### Stable releases

| Prefix | Effect |
|--------|--------|
| `patch:` | Bug fixes and backwards-compatible changes |
| `minor:` | Backwards-compatible new features |
| `major:` | Breaking changes |

### Release candidates

| Prefix | Effect |
|--------|--------|
| `rc-patch:` | Release candidate for a patch version (e.g. `v1.2.4-rc.1`) |
| `rc-minor:` | Release candidate for a minor version (e.g. `v1.3.0-rc.1`) |
| `rc-major:` | Release candidate for a major version (e.g. `v2.0.0-rc.1`) |

RC tags are created as GitHub pre-releases and do not affect the stable version baseline. Each subsequent PR using the same `rc-*` prefix that resolves to the same base version automatically increments the RC counter (`rc.1` → `rc.2` → `rc.3`, etc.).

**Examples:**
- `patch: Fix output variable typo in vnet module`
- `minor: Add support for private endpoint in storage module`
- `major: Upgrade to new major provider version`
- `rc-patch: Fix output variable typo in vnet module (release candidate)`
- `rc-minor: Add support for private endpoint in storage module (release candidate)`
- `rc-major: Upgrade to new major provider version (release candidate)`

If the PR title does not follow this format, the workflow will fail and display an error message with guidance.

## Example Calls

To reuse this workflow in your GitHub repository, reference it in your workflow file using the following pattern:

```yaml
jobs:
  check-pr-title:
    uses: gatesfoundation/terraform-reusable-workflows/.github/workflows/reusable-check-pr-title.yml@main
```

Or, to pin to a specific released version:

```yaml
jobs:
  check-pr-title:
    uses: ac-on-ac/workflow-templates/.github/workflows/reusable-check-pr-title.yml@check-pr-title-v2.0.0 # Replace with desired version
```

## Error Handling

The workflow will fail if:
- The PR title does not start with one of: `patch:`, `minor:`, `major:`, `rc-patch:`, `rc-minor:`, or `rc-major:`

The error message will include examples of all valid PR title formats and instructions for correcting the format.

## Best Practices

1. Always begin your PR title with the appropriate semantic versioning prefix
2. Use `rc-*` prefixes when you want to publish a pre-release for testing before committing to a stable version
3. Use clear and descriptive titles to help automate release workflows
4. Reference this workflow in your repository to enforce PR title standards

---

[Return to the main repository documentation](../README.md)
