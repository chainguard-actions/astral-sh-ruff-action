<!-- markdownlint-disable -->

# Hardening Report: astral-sh--ruff-action/v3.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **astral-sh--ruff-action/v3.5.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags instead of pinned SHA digests, making them vulnerable to supply-chain attacks.

codeql-analysis.yml: actions/checkout@v4, github/codeql-action/init@v3, github/codeql-action/autobuild@v3, github/codeql-action/analyze@v3
release-drafter.yml: release-drafter/release-drafter@v6.1.0
test.yml: actions/checkout@v4 (multiple occurrences), actions/setup-node@v4
update-known-checksums.yml: actions/checkout@v4, actions/setup-node@v4
update-major-minor-tags.yml: actions/checkout@v4

Locations:

- `.github/workflows/codeql-analysis.yml:36`
- `.github/workflows/codeql-analysis.yml:41`
- `.github/workflows/codeql-analysis.yml:52`
- `.github/workflows/codeql-analysis.yml:60`
- `.github/workflows/release-drafter.yml:17`
- `.github/workflows/test.yml:18`
- `.github/workflows/test.yml:22`
- `.github/workflows/update-known-checksums.yml:13`
- `.github/workflows/update-known-checksums.yml:14`
- `.github/workflows/update-major-minor-tags.yml:18`

### missing-permissions (severity: medium)

The workflow file test.yml has no top-level `permissions:` key and none of its jobs define job-level `permissions:` blocks. This means the workflow runs with the default (potentially broad) permissions.

Locations:

- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks in test.yml directly interpolate GitHub Actions expressions (`${{ ... }}`) inside shell commands, violating rule (a). This allows expression values to be interpreted as shell code.

1. `if [ ${{ steps.install.outcome }} == "success" ]` — steps context expression directly in shell
2. `if [ ${{ steps.format-should-fail.outcome }} == "success" ]` — steps context expression directly in shell
3. `if [ ${{ steps.check-should-fail.outcome }} == "success" ]` — steps context expression directly in shell
4. `echo "All jobs passed: ${{ !contains(needs.*.result, 'failure') }}"` — needs context expression directly in shell
5. `exit ${{ contains(needs.*.result, 'failure') && 1 || 0 }}` — needs context expression directly in shell

Locations:

- `.github/workflows/test.yml:63`
- `.github/workflows/test.yml:183`
- `.github/workflows/test.yml:196`
- `.github/workflows/test.yml:228`
- `.github/workflows/test.yml:230`

### script-injection (severity: high)

In update-known-checksums.yml, the `run:` block directly interpolates `${{ secrets.GITHUB_TOKEN }}` as a positional CLI argument in a shell command, violating rule (a). Any `${{ }}` expression directly inside a `run:` block is a script-injection risk. The token should be passed via an `env:` variable instead.

Offending line: `node dist/update-known-checksums/index.js src/download/checksum/known-checksums.ts ${{ secrets.GITHUB_TOKEN }}`

Locations:

- `.github/workflows/update-known-checksums.yml:19`

### github-env-injection (severity: high)

No github-env-injection issues found in action.yml (it uses node20 runner with no run: blocks) or workflow files.

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all findings across 5 workflow files:

1. **unpinned-uses**: Pinned all mutable action references to full commit SHAs:
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5 (in codeql-analysis.yml, test.yml, update-known-checksums.yml, update-major-minor-tags.yml)
   - github/codeql-action/init@v3, autobuild@v3, analyze@v3 → @411c4c9a36b3fca4d674f06b6396b2c6d23522c6 (in codeql-analysis.yml)
   - release-drafter/release-drafter@v6.1.0 → @b1476f6e6eb133afa41ed8589daba6dc69b4d3f5 (in release-drafter.yml)
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 (in test.yml, update-known-checksums.yml)

2. **missing-permissions**: Added `permissions: contents: read` top-level block to test.yml.

3. **script-injection (test.yml)**: Moved all 5 ${{ }} expressions from run: blocks to env: blocks:
   - steps.install.outcome → INSTALL_OUTCOME env var
   - steps.format-should-fail.outcome → FORMAT_OUTCOME env var
   - steps.check-should-fail.outcome → CHECK_OUTCOME env var
   - !contains(needs.*.result, 'failure') → ALL_PASSED env var
   - contains(needs.*.result, 'failure') && 1 || 0 → HAS_FAILURE env var (replaced exit ${{ ... }} with if/then logic)

4. **script-injection (update-known-checksums.yml)**: Moved ${{ secrets.GITHUB_TOKEN }} from positional CLI argument to env: block as GITHUB_TOKEN, referenced as "$GITHUB_TOKEN" in the shell command.

5. **github-env-injection**: Finding noted no issues existed, no changes needed.

