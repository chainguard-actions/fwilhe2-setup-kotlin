<!-- markdownlint-disable -->

# Hardening Report: fwilhe2--setup-kotlin/v1.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fwilhe2--setup-kotlin/v1.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files use mutable tag or branch refs instead of pinned 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if the referenced action is compromised or the tag is moved.

In .github/workflows/Release.yml:
- `uses: actions/checkout@v5` (line 11)
- `uses: fwilhe2/bump-version@main` (line 15)

In .github/workflows/test.yml:
- `uses: actions/checkout@v5` (multiple jobs, e.g. line 14)
- `uses: actions/setup-java@v5` (multiple jobs, e.g. line 15)
- `uses: actions/setup-node@v6` (line 19)
- `uses: typesafegithub/github-actions-typing@v2` (line 116)

All of these should be pinned to their full 40-character commit SHA.

Locations:

- `.github/workflows/Release.yml:11`
- `.github/workflows/Release.yml:15`
- `.github/workflows/test.yml:14`
- `.github/workflows/test.yml:15`
- `.github/workflows/test.yml:19`
- `.github/workflows/test.yml:116`

### missing-permissions (severity: medium)

Neither workflow file defines a top-level `permissions:` key, and no individual job within either file defines a `permissions:` block. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (which may be `write-all` for older repositories), granting unnecessarily broad access to the GITHUB_TOKEN. Both files should declare minimal required permissions.

Locations:

- `.github/workflows/Release.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

In .github/workflows/Release.yml, two `run:` blocks directly interpolate `${{ steps.bump.outputs.newVersion }}` (a `steps.*.outputs.*` context value) into shell command strings. This is a sub-rule (a) violation: the expression is expanded by the Actions template engine before the shell ever sees it, allowing an attacker who can influence the output of the `bump-version` step to inject arbitrary shell commands.

Offending lines:
- Line 18: `run: echo New Version Number ${{ steps.bump.outputs.newVersion }}`
- Line 22: `run: gh release create ${{ steps.bump.outputs.newVersion }} --title "Release ${{ steps.bump.outputs.newVersion }}" --generate-notes`

Fix: assign the value to an environment variable and reference it as a quoted shell variable:
```yaml
env:
  NEW_VERSION: ${{ steps.bump.outputs.newVersion }}
run: gh release create "$NEW_VERSION" --title "Release $NEW_VERSION" --generate-notes
```

Locations:

- `.github/workflows/Release.yml:18`
- `.github/workflows/Release.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across .github/workflows/Release.yml and .github/workflows/test.yml:

1. **unpinned-uses**: Pinned all action references to full 40-char SHAs with tag comments:
   - actions/checkout@v5 → @fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09 # v5
   - fwilhe2/bump-version@main → @c6a73db2f99dd2bc980e85dd15fbdcccac8bc8e5 # main
   - actions/setup-java@v5 → @03ad4de0992f5dab5e18fcb136590ce7c4a0ac95 # v5
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38 # v6
   - typesafegithub/github-actions-typing@v2 → @9ddf35b71a482be7d8922b28e8d00df16b77e315 # v2

2. **missing-permissions**: Added top-level `permissions:` blocks:
   - Release.yml: `contents: write` (needed to create GitHub releases)
   - test.yml: `contents: read` (read-only for checkout/test workflows)

3. **script-injection**: Fixed both `run:` blocks in Release.yml that interpolated `${{ steps.bump.outputs.newVersion }}` directly into shell strings. Moved the value into `env: NEW_VERSION: ${{ steps.bump.outputs.newVersion }}` and referenced it as `"$NEW_VERSION"` in the shell commands.

