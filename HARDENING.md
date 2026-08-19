<!-- markdownlint-disable -->

# Hardening Report: astral-sh--ruff-action/v3.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--ruff-action/v3.5.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions by mutable tags instead of full 40-character SHA digests, making them vulnerable to supply-chain attacks.

.github/workflows/codeql-analysis.yml:
  - uses: actions/checkout@v4 (line 42)
  - uses: github/codeql-action/init@v3 (line 46)
  - uses: github/codeql-action/autobuild@v3 (line 54)
  - uses: github/codeql-action/analyze@v3 (line 66)

.github/workflows/release-drafter.yml:
  - uses: release-drafter/release-drafter@v6.1.0 (line 20)

.github/workflows/test.yml:
  - uses: actions/checkout@v4 (multiple occurrences, e.g. line 17)
  - uses: actions/setup-node@v4 (line 20)

.github/workflows/update-known-checksums.yml:
  - uses: actions/checkout@v4 (line 14)
  - uses: actions/setup-node@v4 (line 15)

.github/workflows/update-major-minor-tags.yml:
  - uses: actions/checkout@v4 (line 19)

Note: eifinger/actionlint-action@23c85443d840cd73bbecb9cddfc933cc21649a38 in test.yml is correctly SHA-pinned and peter-evans/create-pull-request@271a8d0340265f705b14b6d32b9829c1cb33d45e in update-known-checksums.yml is also correctly SHA-pinned.

Locations:

- `.github/workflows/codeql-analysis.yml:42`
- `.github/workflows/codeql-analysis.yml:46`
- `.github/workflows/codeql-analysis.yml:54`
- `.github/workflows/codeql-analysis.yml:66`
- `.github/workflows/release-drafter.yml:20`
- `.github/workflows/test.yml:17`
- `.github/workflows/test.yml:20`
- `.github/workflows/update-known-checksums.yml:14`
- `.github/workflows/update-known-checksums.yml:15`
- `.github/workflows/update-major-minor-tags.yml:19`

### script-injection (severity: high)

GitHub Actions expressions are interpolated directly inside run: shell command strings (sub-rule a), allowing YAML template substitution to inject arbitrary shell content before the shell parses the command.

.github/workflows/test.yml:
  - `if [ ${{ steps.install.outcome }} == "success" ]; then` — steps context expression in run: block
  - `if [ ${{ steps.format-should-fail.outcome }} == "success" ]; then` — steps context expression in run: block
  - `if [ ${{ steps.check-should-fail.outcome }} == "success" ]; then` — steps context expression in run: block
  - `echo "All jobs passed: ${{ !contains(needs.*.result, 'failure') }}"` — needs context expression in run: block
  - `exit ${{ contains(needs.*.result, 'failure') && 1 || 0 }}` — needs context expression in run: block

.github/workflows/update-known-checksums.yml:
  - `src/download/checksum/known-checksums.ts ${{ secrets.GITHUB_TOKEN }}` — secrets expression passed directly as a shell argument in a run: block

All of these should use environment variables instead of direct expression interpolation.

Locations:

- `.github/workflows/test.yml:71`
- `.github/workflows/test.yml:270`
- `.github/workflows/test.yml:280`
- `.github/workflows/test.yml:320`
- `.github/workflows/test.yml:322`
- `.github/workflows/update-known-checksums.yml:22`

### missing-permissions (severity: medium)

The workflow file .github/workflows/test.yml has no top-level `permissions:` key and none of its jobs define job-level `permissions:` blocks. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to contents by default on many repositories). Explicit minimal permissions should be declared.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all findings across 5 workflow files:

1. **unpinned-uses**: Pinned all mutable action references to full SHA digests:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 (19 occurrences across 4 files)
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 (2 occurrences)
   - github/codeql-action/{init,autobuild,analyze}@v3 → @f3712979fa5f215279b101dd0a2e3bdfb4353324
   - release-drafter/release-drafter@v6.1.0 → @b1476f6e6eb133afa41ed8589daba6dc69b4d3f5

2. **script-injection**: Moved all ${{ }} expressions from run: shell strings to env: blocks:
   - test.yml: steps.install.outcome, steps.format-should-fail.outcome, steps.check-should-fail.outcome, and needs context expressions (all-tests-passed job)
   - update-known-checksums.yml: secrets.GITHUB_TOKEN passed as shell argument

3. **missing-permissions**: Added `permissions: contents: read` at the top level of test.yml

