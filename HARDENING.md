<!-- markdownlint-disable -->

# Hardening Report: endorama--asdf-parse-tool-versions/v1.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **endorama--asdf-parse-tool-versions/v1.5.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Three `run:` steps in test.yml directly interpolate `${{ }}` expressions into shell command strings without routing through env vars. (1) Line 24: `run: echo "${{ env.NODEJS_VERSION }}"` — `env.*` context injected directly. (2) Line 25: `run: echo "${{ fromJSON(steps.versions.outputs.tools).nodejs }}"` — `steps.*.outputs.*` context injected directly. (3) Line 34: `run: ./.github/workflows/scripts/test/test-json.sh '${{ steps.custom_versions_file.outputs.tools }}'` — `steps.*.outputs.*` context injected directly as a shell argument. Any of these values could contain shell metacharacters that execute arbitrary commands.

Locations:

- `.github/workflows/test.yml:24`
- `.github/workflows/test.yml:25`
- `.github/workflows/test.yml:34`

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files use mutable tag-based refs instead of immutable 40-character SHA digests, making them vulnerable to supply-chain attacks if the upstream tag is moved or compromised. Failing references: check-dist.yml — `actions/checkout@v6`, `actions/setup-node@v6`, `actions/upload-artifact@v5`; test.yml — `actions/checkout@v6` (used twice).

Locations:

- `.github/workflows/check-dist.yml:18`
- `.github/workflows/check-dist.yml:21`
- `.github/workflows/check-dist.yml:42`
- `.github/workflows/test.yml:13`
- `.github/workflows/test.yml:21`

### missing-permissions (severity: medium)

Neither `check-dist.yml` nor `test.yml` declares a top-level `permissions:` block, and no individual job within either file declares its own `permissions:` block. Without explicit permissions, workflows inherit the repository's default token permissions (often `write-all`), granting unnecessarily broad access. Each workflow should declare minimal required permissions (e.g., `contents: read`).

Locations:

- `.github/workflows/check-dist.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across .github/workflows/test.yml and .github/workflows/check-dist.yml:

1. **script-injection** (test.yml lines 24, 25, 34): Moved all three `${{ }}` expressions out of `run:` shell strings into `env:` blocks. (a) `env.NODEJS_VERSION` → env var `NODEJS_VERSION`; (b) `fromJSON(steps.versions.outputs.tools).nodejs` → env var `NODEJS_FROM_OUTPUT`; (c) `steps.custom_versions_file.outputs.tools` passed as shell argument → env var `TOOLS_OUTPUT` referenced as `"$TOOLS_OUTPUT"`.

2. **unpinned-uses**: Pinned all five mutable tag references to full 40-char SHAs with tag comments: `actions/checkout@v6` → `@df4cb1c069e1874edd31b4311f1884172cec0e10 # v6`; `actions/setup-node@v6` → `@249970729cb0ef3589644e2896645e5dc5ba9c38 # v6`; `actions/upload-artifact@v5` → `@330a01c490aca151604b8cf639adc76d48f6c5d4 # v5`.

3. **missing-permissions**: Added `permissions: contents: read` top-level block to both check-dist.yml and test.yml.

