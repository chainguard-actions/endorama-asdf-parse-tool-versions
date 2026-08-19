<!-- markdownlint-disable -->

# Hardening Report: endorama--asdf-parse-tool-versions/v1.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **endorama--asdf-parse-tool-versions/v1.4.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Both workflow files reference GitHub Actions using mutable version tags instead of pinned 40-character SHA commit hashes. In check-dist.yml: `actions/checkout@v5`, `actions/setup-node@v6`, `actions/upload-artifact@v5`. In test.yml: `actions/checkout@v5`. These tags can be moved to point to different (potentially malicious) commits at any time, enabling supply-chain attacks.

Locations:

- `.github/workflows/check-dist.yml:21`
- `.github/workflows/check-dist.yml:24`
- `.github/workflows/check-dist.yml:44`
- `.github/workflows/test.yml:13`

### missing-permissions (severity: medium)

Neither workflow file defines a top-level `permissions:` block, and no individual job within them defines `permissions:` either. Without explicit permissions, workflows run with the default (potentially broad) GITHUB_TOKEN permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/check-dist.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

In test.yml, three `run:` steps interpolate GitHub Actions expressions directly into shell command strings (sub-rule a), which causes the expression value to be substituted into the shell command before execution. This allows an attacker to inject arbitrary shell commands via controlled values: (1) `run: echo "${{ env.NODEJS_VERSION }}"` — env context interpolated directly; (2) `run: echo "${{ fromJSON(steps.versions.outputs.tools).nodejs }}"` — steps output interpolated directly; (3) `run: ./.github/workflows/scripts/test/test-json.sh '${{ steps.custom_versions_file.outputs.tools }}'` — steps output interpolated directly into a shell argument. All three should be routed through env: variables and referenced as quoted shell variables instead.

Locations:

- `.github/workflows/test.yml:26`
- `.github/workflows/test.yml:27`
- `.github/workflows/test.yml:34`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across both workflow files:

1. **unpinned-uses**: Pinned all action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v5 → @93cb6efe18208431cddfb8368fd83d5badbf9bfd (both files)
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38 (check-dist.yml)
   - actions/upload-artifact@v5 → @330a01c490aca151604b8cf639adc76d48f6c5d4 (check-dist.yml)

2. **missing-permissions**: Added `permissions: contents: read` top-level block to both check-dist.yml and test.yml.

3. **script-injection**: Fixed all three injection points in test.yml by moving expressions into `env:` blocks and referencing them as quoted shell variables:
   - `${{ env.NODEJS_VERSION }}` → env var NODEJS_VERSION
   - `${{ fromJSON(steps.versions.outputs.tools).nodejs }}` → env var NODEJS_OUTPUT
   - `${{ steps.custom_versions_file.outputs.tools }}` → env var TOOLS_OUTPUT (passed as "$TOOLS_OUTPUT" to the test script)

