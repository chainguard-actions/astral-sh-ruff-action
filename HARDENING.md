<!-- markdownlint-disable -->

# Hardening Report: astral-sh--ruff-action/v3.5.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--ruff-action/v3.5.1** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside `run:` shell command strings in test.yml. Specifically: `if [ ${{ steps.install.outcome }} == "success" ]`, `if [ ${{ steps.format-should-fail.outcome }} == "success" ]`, `if [ ${{ steps.check-should-fail.outcome }} == "success" ]`, and `exit ${{ contains(needs.*.result, 'failure') && 1 || 0 }}`. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it, allowing metacharacter injection.

Locations:

- `.github/workflows/test.yml:57`
- `.github/workflows/test.yml:238`
- `.github/workflows/test.yml:249`
- `.github/workflows/test.yml:263`

### unpinned-uses (severity: high)

Multiple workflow files reference actions by mutable tag or version string instead of a full 40-character commit SHA. Unpinned references are vulnerable to supply-chain attacks if the tag is moved. Failing references include: test.yml — `actions/checkout@v5` (multiple steps), `actions/setup-node@v4`; codeql-analysis.yml — `actions/checkout@v5`, `github/codeql-action/init@v3`, `github/codeql-action/autobuild@v3`, `github/codeql-action/analyze@v3`; release-drafter.yml — `release-drafter/release-drafter@v6.1.0`; update-known-checksums.yml — `actions/checkout@v5`, `actions/setup-node@v4`; update-major-minor-tags.yml — `actions/checkout@v5`.

Locations:

- `.github/workflows/test.yml:18`
- `.github/workflows/test.yml:21`
- `.github/workflows/codeql-analysis.yml:34`
- `.github/workflows/codeql-analysis.yml:38`
- `.github/workflows/codeql-analysis.yml:47`
- `.github/workflows/codeql-analysis.yml:62`
- `.github/workflows/release-drafter.yml:17`
- `.github/workflows/update-known-checksums.yml:13`
- `.github/workflows/update-known-checksums.yml:14`
- `.github/workflows/update-major-minor-tags.yml:14`

### missing-permissions (severity: medium)

The workflow file test.yml has no top-level `permissions:` key and none of its jobs define a `permissions:` block. Without explicit permissions, the workflow inherits the default repository permissions (which may include write access to contents and other scopes), violating the principle of least privilege.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across 5 workflow files:

1. script-injection (test.yml): Moved all 4 ${{ }} expressions out of run: shell strings into env: blocks. The outcome checks now use env vars (INSTALL_OUTCOME, FORMAT_OUTCOME, CHECK_OUTCOME). The complex contains(needs.*.result, 'failure') expression was replaced with NEEDS_RESULTS=${{ toJSON(needs.*.result) }} and grep-based shell logic.

2. unpinned-uses: Pinned all mutable action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v5 → @93cb6efe18208431cddfb8368fd83d5badbf9bfd (test.yml, codeql-analysis.yml, update-known-checksums.yml, update-major-minor-tags.yml)
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 (test.yml, update-known-checksums.yml)
   - github/codeql-action/{init,autobuild,analyze}@v3 → @b7351df727350dca84cb9d725d57dcf5bc82ba26 (codeql-analysis.yml)
   - release-drafter/release-drafter@v6.1.0 → @b1476f6e6eb133afa41ed8589daba6dc69b4d3f5 (release-drafter.yml)

3. missing-permissions (test.yml): Added top-level `permissions: contents: read` block to enforce least-privilege.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/update-known-checksums.yml line 22. Moved `${{ secrets.GITHUB_TOKEN }}` out of the `run:` shell command and into an `env:` block as `TOKEN: ${{ secrets.GITHUB_TOKEN }}`. The shell command now references it as `"$TOKEN"` instead of directly interpolating the expression.

