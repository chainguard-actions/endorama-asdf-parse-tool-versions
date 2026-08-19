<!-- markdownlint-disable -->

# Hardening Report: endorama--asdf-parse-tool-versions/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **endorama--asdf-parse-tool-versions/v1.4.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Both workflow files reference actions using mutable version tags instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks if the tag is moved.

check-dist.yml:
  - actions/checkout@v5 (line 23)
  - actions/setup-node@v6 (line 26)
  - actions/upload-artifact@v5 (line 48)

test.yml:
  - actions/checkout@v5 (line 13)
  - actions/checkout@v5 (line 21)

Locations:

- `.github/workflows/check-dist.yml:23`
- `.github/workflows/check-dist.yml:26`
- `.github/workflows/check-dist.yml:48`
- `.github/workflows/test.yml:13`
- `.github/workflows/test.yml:21`

### permissions (severity: medium)

Neither workflow file has a top-level `permissions:` key, and no individual job defines its own `permissions:` block. This means jobs run with the default (potentially broad) GITHUB_TOKEN permissions. Both check-dist.yml and test.yml are affected.

Locations:

- `.github/workflows/check-dist.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Three `run:` steps in test.yml directly interpolate `${{ ... }}` expressions inside shell command strings (rule a). GitHub Actions performs template substitution before the shell parses the string, allowing an attacker-controlled value to inject arbitrary shell commands.

- Line 25: `run: echo "${{ env.NODEJS_VERSION }}"` — env.* context interpolated directly into shell.
- Line 26: `run: echo "${{ fromJSON(steps.versions.outputs.tools).nodejs }}"` — steps.*.outputs.* interpolated directly into shell.
- Line 34: `run: ./.github/workflows/scripts/test/test-json.sh '${{ steps.custom_versions_file.outputs.tools }}'` — steps.*.outputs.* interpolated directly as a shell argument.

Fix: move the values into `env:` variables and reference them as quoted shell variables (e.g., `"$ENV_VAR"`).

Locations:

- `.github/workflows/test.yml:25`
- `.github/workflows/test.yml:26`
- `.github/workflows/test.yml:34`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings across both workflow files:

1. **unpinned-uses**: Pinned all 5 action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v5 → fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09 (both files)
   - actions/setup-node@v6 → 249970729cb0ef3589644e2896645e5dc5ba9c38 (check-dist.yml)
   - actions/upload-artifact@v5 → 330a01c490aca151604b8cf639adc76d48f6c5d4 (check-dist.yml)

2. **permissions**: Added `permissions: {}` top-level block to both check-dist.yml and test.yml to enforce least-privilege GITHUB_TOKEN access.

3. **script-injection**: Moved all three ${{ }} expressions in test.yml run: steps into env: blocks and referenced them as quoted shell variables ($NODEJS_VERSION, $NODEJS_OUTPUT, $TOOLS_OUTPUT), preventing attacker-controlled values from being interpreted as shell commands.

