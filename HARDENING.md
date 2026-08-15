<!-- markdownlint-disable -->

# Hardening Report: astral-sh--ruff-action/v3.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--ruff-action/v3.6.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a `run:` shell command string. In `update-known-checksums.yml`, the run block passes `${{ secrets.GITHUB_TOKEN }}` directly as a command-line argument: `node dist/update-known-checksums/index.js src/download/checksum/known-checksums.ts ${{ secrets.GITHUB_TOKEN }}`. Any `${{ ... }}` expression inside a `run:` block is a script injection risk because YAML template substitution happens before the shell ever sees the value. The token should be passed via an `env:` variable and referenced as `$ENV_VAR` in the shell script.

Locations:

- `.github/workflows/update-known-checksums.yml:24`

### script-injection (severity: high)

Sub-rule (a): Two `${{ ... }}` expressions are directly interpolated inside a `run:` shell command string in the `all-tests-passed` job of `test.yml`. The offending lines are: `echo "All jobs passed: ${{ !(contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) }}"` and `exit ${{ (contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) && 1 || 0 }}`. These expressions are substituted by the YAML template engine before the shell executes them, bypassing shell quoting. They should be moved to `env:` variables and referenced as shell variables.

Locations:

- `.github/workflows/test.yml:338`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection findings:
1. `.github/workflows/update-known-checksums.yml` (line 24): Moved `${{ secrets.GITHUB_TOKEN }}` from a direct command-line argument in the `run:` block to an `env:` variable (`GITHUB_TOKEN`), referenced as `"$GITHUB_TOKEN"` in the shell command.
2. `.github/workflows/test.yml` (line 338): Moved two `${{ ... }}` expressions in the `all-tests-passed` job's `run:` block to `env:` variables (`ALL_PASSED` and `EXIT_CODE`), referenced as `$ALL_PASSED` and `"$EXIT_CODE"` in the shell script. This prevents YAML template substitution from injecting values directly into shell commands before the shell executes them.

