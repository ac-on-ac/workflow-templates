# Releasing Individual Workflows

This document describes the process for releasing and versioning individual workflows in the workflow-templates repository. Versioning workflows allows consumers to reference specific, stable versions of each workflow in their CI/CD pipelines.

## The Versioning System

Workflows in this repository follow semantic versioning principles:

- **Major version**: Incremented for breaking changes that require consumers to modify their implementation
- **Minor version**: Incremented for backward-compatible feature additions
- **Patch version**: Incremented for backward-compatible bug fixes
- **Release candidates**: Pre-release versions for testing before committing to a stable version

Each workflow is versioned independently with a tag in the format:

```
<workflow-base-name>-v<major>.<minor>.<patch>
```

For example:
- `terraform-test-v1.0.0`
- `terraform-docs-v2.3.1`

Release candidate tags follow the format:

```
<workflow-base-name>-v<major>.<minor>.<patch>-rc.<n>
```

For example:
- `terraform-test-v2.0.0-rc.1`
- `terraform-docs-v2.3.1-rc.2`

## Release Process

The repository includes an automated workflow called "Release Single Workflow" that handles the versioning and release process.

### Prerequisites

Before releasing a workflow, ensure that:

1. The workflow has been properly tested
2. Any major changes are clearly documented
3. An example file exists in the `docs/examples/` directory (optional but recommended)

### Steps to Release a Workflow

1. **Navigate to the Actions tab** of the workflow-templates repository
2. **Select the "Release Single Workflow"** workflow from the list
3. **Click "Run workflow"**
4. Fill in the required information:
   - **Workflow file name**: The exact filename of the workflow to release (e.g. `reusable-terraform-test.yml`)
   - **Version type**: Choose based on the nature of the changes:

| Option | Creates | Use when |
|--------|---------|----------|
| `patch` | `vX.Y.Z+1` | Backwards-compatible bug fixes |
| `minor` | `vX.Y+1.0` | Backwards-compatible new features |
| `major` | `vX+1.0.0` | Breaking changes |
| `rc-patch` | `vX.Y.Z+1-rc.N` | Pre-release for an upcoming patch |
| `rc-minor` | `vX.Y+1.0-rc.N` | Pre-release for an upcoming minor version |
| `rc-major` | `vX+1.0.0-rc.N` | Pre-release for an upcoming major version |

5. **Click "Run workflow"** to start the process

### What the Release Process Does

The release process automatically:

1. Verifies that the specified workflow file exists
2. Checks for an example file in `docs/examples/` with the same name
3. Calculates the new version number based on:
   - The latest existing **stable** tag for the workflow (RC tags are excluded from the baseline)
   - The selected version type
   - For RC versions: finds the highest existing RC tag for the calculated base version and increments the counter, or starts at `rc.1`
4. Creates a Git tag using the format `<workflow-base-name>-v<version>`
5. Creates or updates a GitHub Release with:
   - The same name as the tag
   - Release notes that include the example workflow call (if available)
   - Marked as a **pre-release** for RC tags; marked as a full release for stable tags

### Initial Release

For the first release of a workflow, selecting `patch`, `minor`, or `major` will create `v1.0.0`. Selecting `rc-patch`, `rc-minor`, or `rc-major` will create `v1.0.0-rc.1`.

### Release Candidate Workflow

A typical RC workflow looks like:

1. Run with `rc-minor` → creates `terraform-test-v1.1.0-rc.1` (pre-release)
2. Test with consumers; fix issues; run again with `rc-minor` → creates `terraform-test-v1.1.0-rc.2`
3. When satisfied, run with `minor` → creates `terraform-test-v1.1.0` (full release)

## Using Versioned Workflows

Once a workflow has been released with a version tag, consumers can reference that specific version in their GitHub Actions workflows.

### Referencing the Latest Version

```yaml
jobs:
  my-job:
    uses: ac-on-ac/workflow-templates/.github/workflows/reusable-terraform-test.yml@main
```

### Referencing a Specific Stable Version

```yaml
jobs:
  my-job:
    uses: ac-on-ac/workflow-templates/.github/workflows/reusable-terraform-test.yml@terraform-test-v1.2.3
```

### Referencing a Release Candidate

```yaml
jobs:
  my-job:
    uses: ac-on-ac/workflow-templates/.github/workflows/reusable-terraform-test.yml@terraform-test-v2.0.0-rc.1
```

## Example Files

It is highly recommended to include an example file showing how to call your reusable workflow. This makes it easier for users to understand the required inputs and how to use the workflow.

Create your example file in the `docs/examples/` directory with the same name as your workflow file. For example:

- Workflow: `.github/workflows/reusable-terraform-test.yml`
- Example: `docs/examples/reusable-terraform-test.yml`

The example file will be automatically included in the GitHub Release notes.

## Best Practices

1. **Make semantic version choices thoughtfully**:
   - Use `patch` for bug fixes and minor improvements
   - Use `minor` for new features that don't break existing usage
   - Use `major` for breaking changes
   - Use `rc-*` variants to validate changes with real consumers before publishing a stable version

2. **Document your changes**:
   - Keep examples up to date
   - Update documentation when workflow behaviour changes

3. **Test before releasing**:
   - Ensure the workflow functions as expected before creating a release
   - Consider using an RC release to test with consumers before promoting to stable

4. **Communicate breaking changes**:
   - When releasing a major version, clearly document the breaking changes
   - Consider maintaining older versions with critical bug fixes

---

[Return to the main repository documentation](../README.md)
