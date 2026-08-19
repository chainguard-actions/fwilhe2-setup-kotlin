<!-- markdownlint-disable -->

# Hardening Report: fwilhe2--setup-kotlin/v1.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fwilhe2--setup-kotlin/v1.8** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses 'fwilhe2/bump-version@main', which is pinned to a mutable branch ref ('main') rather than a full 40-character commit SHA. This means the action can be silently updated or replaced with malicious code without any change to the workflow file.

Locations:

- `.github/workflows/Release.yml:13`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings. In Release.yml, '${{ steps.bump.outputs.newVersion }}' is embedded directly in two run: blocks. The value of steps.bump.outputs.newVersion is workflow-controllable and flows through YAML template substitution before the shell sees it, enabling command injection. Offending lines: (1) 'run: echo New Version Number ${{ steps.bump.outputs.newVersion }}' and (2) 'run: gh release create ${{ steps.bump.outputs.newVersion }} --title "Release ${{ steps.bump.outputs.newVersion }}" --generate-notes'.

Locations:

- `.github/workflows/Release.yml:15`
- `.github/workflows/Release.yml:18`

### missing-permissions (severity: medium)

The workflow file has no top-level 'permissions:' key and no job-level 'permissions:' key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g., write access to contents). Explicit minimal permissions should be declared.

Locations:

- `.github/workflows/Release.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level 'permissions:' key and no job-level 'permissions:' key on any of its jobs (build, test, test-with-arg, test-with-arg-default-version, test-with-native, validate-typings). Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. Explicit minimal permissions should be declared.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed Release.yml: (1) Pinned fwilhe2/bump-version@main to full SHA c6a73db2f99dd2bc980e85dd15fbdcccac8bc8e5 with '# main' comment. (2) Moved both ${{ steps.bump.outputs.newVersion }} expressions out of run: blocks into env: blocks as NEW_VERSION, referencing $NEW_VERSION in the shell scripts. (3) Added top-level 'permissions: contents: write' (needed for gh release create). Fixed test.yml: Added top-level 'permissions: contents: read' (sufficient for checkout and testing operations).

### Iteration 2

**Notes:**

The script-injection finding was a false positive — the analysis agent's own description concludes 'No script-injection finding applies here.' The update_dist.yaml workflow already correctly places `inputs.branch` into an `env:` block as `TARGET_BRANCH` and uses it with proper double-quoting (`"${TARGET_BRANCH}"`) in the shell command. No changes were required.

