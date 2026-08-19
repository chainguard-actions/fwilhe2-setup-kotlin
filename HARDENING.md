<!-- markdownlint-disable -->

# Hardening Report: fwilhe2--setup-kotlin/v1.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fwilhe2--setup-kotlin/v1.7** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses `fwilhe2/bump-version@main`, which is pinned to a mutable branch name rather than a full 40-character commit SHA. This means the action can be silently updated (or compromised) without any change to the workflow file, creating a supply-chain risk.

Locations:

- `.github/workflows/Release.yml:15`

### script-injection (severity: high)

Two `run:` steps in Release.yml directly interpolate `${{ steps.bump.outputs.newVersion }}` into shell commands (sub-rule a). `steps.*.outputs.*` is a workflow-controllable context that is substituted by the YAML template engine before the shell ever sees it, allowing an attacker who can influence the step output to inject arbitrary shell commands.

Offending lines:
- `run: echo New Version Number ${{ steps.bump.outputs.newVersion }}`
- `run: gh release create ${{ steps.bump.outputs.newVersion }} --title "Release ${{ steps.bump.outputs.newVersion }}" --generate-notes`

Fix: move the value into an `env:` variable and reference it as a quoted shell variable, e.g. `"$NEW_VERSION"`.

Locations:

- `.github/workflows/Release.yml:18`
- `.github/workflows/Release.yml:21`

### missing-permissions (severity: medium)

Neither `Release.yml` nor `test.yml` declares a top-level `permissions:` key, and none of their jobs declares a job-level `permissions:` key. Without explicit permissions, GitHub Actions grants the default token permissions (which can be broad depending on repository settings), violating the principle of least privilege. Add a top-level `permissions: {}` block and grant only the specific scopes each workflow requires.

Locations:

- `.github/workflows/Release.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed Release.yml: (1) Pinned fwilhe2/bump-version@main to full SHA c6a73db2f99dd2bc980e85dd15fbdcccac8bc8e5; (2) Moved both ${{ steps.bump.outputs.newVersion }} expressions into env: blocks as NEW_VERSION and referenced as "$NEW_VERSION" in shell; (3) Added top-level permissions: contents: write (needed for gh release create). Fixed test.yml: Added top-level permissions: contents: read (minimum needed for checkout and test runs).

