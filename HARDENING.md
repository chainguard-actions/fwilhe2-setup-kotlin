<!-- markdownlint-disable -->

# Hardening Report: fwilhe2--setup-kotlin/v2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fwilhe2--setup-kotlin/v2.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct GitHub Actions expression interpolation inside run: shell commands in Release.yml. Sub-rule (a): ${{ steps.bump.outputs.newVersion }} is interpolated directly into shell commands, allowing an attacker who can influence step outputs to inject arbitrary shell commands. Line 23: `run: echo New Version Number ${{ steps.bump.outputs.newVersion }}`. Line 27: `gh release create ${{ steps.bump.outputs.newVersion }} --title "Release ${{ steps.bump.outputs.newVersion }}" --generate-notes`. These values should be passed via env: variables and then referenced as quoted shell variables (e.g., "$NEW_VERSION").

Locations:

- `.github/workflows/Release.yml:23`
- `.github/workflows/Release.yml:27`

### unpinned-uses (severity: high)

The action reference `fwilhe2/bump-version@main` uses a mutable branch name (@main) instead of a full 40-character commit SHA. This means the action can be silently updated or compromised without any change to the workflow file, creating a supply-chain risk.

Locations:

- `.github/workflows/Release.yml:18`

### missing-permissions (severity: medium)

Release.yml has no top-level `permissions:` key and no job-level `permissions:` key on its only job (`build`). Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. Explicit minimal permissions (e.g., `contents: write` for creating releases) should be declared.

Locations:

- `.github/workflows/Release.yml:1`

### missing-permissions (severity: medium)

test.yml has no top-level `permissions:` key and none of its jobs (build, test, test-with-arg, test-with-arg-default-version, test-with-native, validate-typings) declare a `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions. Explicit minimal permissions (e.g., `contents: read`) should be declared.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings in Release.yml and test.yml:
1. script-injection: Moved both occurrences of ${{ steps.bump.outputs.newVersion }} into env: blocks as NEW_VERSION, then referenced as "$NEW_VERSION" in shell commands (echo and gh release create).
2. unpinned-uses: Pinned fwilhe2/bump-version@main to full SHA 76445810c45e953d0e96b8128748a8f841415d03 with # main comment.
3. missing-permissions (Release.yml): Added top-level `permissions: contents: write` required for creating GitHub releases.
4. missing-permissions (test.yml): Added top-level `permissions: contents: read` as the minimal permission needed for the test workflow.

