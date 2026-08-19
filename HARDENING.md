<!-- markdownlint-disable -->

# Hardening Report: endorama--asdf-parse-tool-versions/v1.5.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **endorama--asdf-parse-tool-versions/v1.5.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All external action references in both workflow files use mutable version tags (@v6) instead of immutable 40-character commit SHA hashes. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit. Affected references: actions/checkout@v6 (test.yml lines 13, 21; check-dist.yml line 22), actions/setup-node@v6 (check-dist.yml line 25), actions/upload-artifact@v6 (check-dist.yml line 46).

Locations:

- `.github/workflows/test.yml:13`
- `.github/workflows/test.yml:21`
- `.github/workflows/check-dist.yml:22`
- `.github/workflows/check-dist.yml:25`
- `.github/workflows/check-dist.yml:46`

### permissions (severity: medium)

missing-permissions: Neither .github/workflows/test.yml nor .github/workflows/check-dist.yml defines a top-level `permissions:` block, and no individual job within either file defines job-level permissions. Without explicit permissions, workflows inherit the default repository permissions (which may include write access), violating the principle of least privilege.

Locations:

- `.github/workflows/test.yml:1`
- `.github/workflows/check-dist.yml:6`

### script-injection (severity: high)

Rule (a) violation: Three `run:` steps in test.yml directly interpolate GitHub Actions expressions into shell command strings without routing through env: variables. (1) Line 24: `run: echo "${{ env.NODEJS_VERSION }}"` — the env context value is interpolated directly into the shell command. (2) Line 25: `run: echo "${{ fromJSON(steps.versions.outputs.tools).nodejs }}"` — step output is interpolated directly. (3) Line 33: `run: ./.github/workflows/scripts/test/test-json.sh '${{ steps.custom_versions_file.outputs.tools }}'` — step output is interpolated directly as a shell argument. Any of these values could contain shell metacharacters that execute arbitrary commands.

Locations:

- `.github/workflows/test.yml:24`
- `.github/workflows/test.yml:25`
- `.github/workflows/test.yml:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings across both workflow files:

1. **unpinned-uses**: Pinned all 5 action references to full 40-char SHAs with tag comments:
   - actions/checkout@v6 → @df4cb1c069e1874edd31b4311f1884172cec0e10 # v6 (both test.yml lines 13, 21 and check-dist.yml line 22)
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38 # v6 (check-dist.yml line 25)
   - actions/upload-artifact@v6 → @b7c566a772e6b6bfb58ed0dc250532a479d7789f # v6 (check-dist.yml line 46)

2. **permissions**: Added `permissions: {}` top-level block to both test.yml and check-dist.yml.

3. **script-injection**: Moved all three ${{ }} expressions out of run: shell strings into env: blocks:
   - `echo "${{ env.NODEJS_VERSION }}"` → env: NODEJS_VERSION + `echo "$NODEJS_VERSION"`
   - `echo "${{ fromJSON(steps.versions.outputs.tools).nodejs }}"` → env: NODEJS_FROM_OUTPUT + `echo "$NODEJS_FROM_OUTPUT"`
   - `test-json.sh '${{ steps.custom_versions_file.outputs.tools }}'` → env: TOOLS_OUTPUT + `test-json.sh "$TOOLS_OUTPUT"`

