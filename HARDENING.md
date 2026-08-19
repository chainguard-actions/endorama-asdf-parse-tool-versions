<!-- markdownlint-disable -->

# Hardening Report: endorama--asdf-parse-tool-versions/v1.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **endorama--asdf-parse-tool-versions/v1.6.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Both workflow files reference external actions using mutable version tags instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if a tag is moved or hijacked.

In .github/workflows/check-dist.yml:
- `uses: actions/checkout@v6` (line 23)
- `uses: actions/setup-node@v6` (line 26)
- `uses: actions/upload-artifact@v7` (line 47)

In .github/workflows/test.yml:
- `uses: actions/checkout@v6` (lines 13, 24, 35)
- `uses: actions/setup-node@v6` (line 15)

All of these should be pinned to their full SHA digest, e.g. `actions/checkout@<40-hex-char-sha> # v6`.

Locations:

- `.github/workflows/check-dist.yml:23`
- `.github/workflows/check-dist.yml:26`
- `.github/workflows/check-dist.yml:47`
- `.github/workflows/test.yml:13`
- `.github/workflows/test.yml:15`
- `.github/workflows/test.yml:24`
- `.github/workflows/test.yml:35`

### missing-permissions (severity: medium)

Neither workflow file defines a top-level `permissions:` block, and no individual job within them defines a `permissions:` block. Without explicit permissions, GitHub Actions grants the default token permissions (which may include write access to repository contents, packages, etc.), violating the principle of least privilege.

Locations:

- `.github/workflows/check-dist.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

In .github/workflows/test.yml, two `run:` steps directly interpolate `${{ steps.*.outputs.* }}` expressions into shell command strings (sub-rule a). Although `steps.*.outputs.*` values originate from the action under test, they are workflow-controllable context values that flow through YAML template substitution before the shell sees them. If the action ever emits attacker-influenced output (e.g. from a tool version string read from a file), this becomes a command injection vector.

Offending lines:
- Line 32: `run: ./.github/workflows/scripts/test/test-json.sh '${{ steps.versions.outputs.tools }}'`
- Line 43: `run: ./.github/workflows/scripts/test/test-json.sh '${{ steps.custom_versions_file.outputs.tools }}'`

Fix: move the value into an `env:` variable and pass the quoted env var to the script instead of interpolating the expression directly.

Locations:

- `.github/workflows/test.yml:32`
- `.github/workflows/test.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across .github/workflows/check-dist.yml and .github/workflows/test.yml:
1. unpinned-uses: Pinned actions/checkout@v6 → SHA d23441a48e516b6c34aea4fa41551a30e30af803, actions/setup-node@v6 → SHA 249970729cb0ef3589644e2896645e5dc5ba9c38, actions/upload-artifact@v7 → SHA 043fb46d1a93c77aae656e7c1c64a875d1fc6a0a. Original tags preserved as inline comments.
2. missing-permissions: Added `permissions: {}` top-level block to both workflow files.
3. script-injection: Moved ${{ steps.versions.outputs.tools }} and ${{ steps.custom_versions_file.outputs.tools }} out of run: shell strings into env: blocks as TOOLS_OUTPUT, then referenced as "$TOOLS_OUTPUT" in the shell commands.

