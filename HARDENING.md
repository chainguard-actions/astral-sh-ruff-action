<!-- markdownlint-disable -->

# Hardening Report: astral-sh--ruff-action/v3.6.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--ruff-action/v3.6.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A `${{ ... }}` expression is directly interpolated inside a `run:` shell command string. In `update-known-checksums.yml`, the `run:` block passes `${{ secrets.GITHUB_TOKEN }}` as a command-line argument directly in the shell script: `src/download/checksum/known-checksums.ts ${{ secrets.GITHUB_TOKEN }}`. Even though `secrets.GITHUB_TOKEN` is not attacker-controlled, any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection finding — the value flows through YAML template substitution before the shell ever sees it.

Locations:

- `.github/workflows/update-known-checksums.yml:24`

### script-injection (severity: high)

Sub-rule (a): `${{ ... }}` expressions are directly interpolated inside `run:` shell command strings in the `all-tests-passed` job. The offending lines are: `echo "All jobs passed: ${{ !(contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) }}"` and `exit ${{ (contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) && 1 || 0 }}`. These `needs.*.result` expressions flow through YAML template substitution before the shell executes them, making them script-injection risks.

Locations:

- `.github/workflows/test.yml:358`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings:
1. `.github/workflows/update-known-checksums.yml` line 24: Moved `${{ secrets.GITHUB_TOKEN }}` out of the `run:` block into an `env:` block as `GITHUB_TOKEN`, then referenced it as `"$GITHUB_TOKEN"` in the shell command.
2. `.github/workflows/test.yml` line 358 (all-tests-passed job): Moved both `${{ needs.*.result }}` expressions out of the `run:` block into an `env:` block as `ALL_PASSED` and `ANY_FAILED`, then referenced them as plain environment variables in the shell script. The `ANY_FAILED` expression evaluates to `'1'` or `'0'` via the ternary-style `&& '1' || '0'` so `exit "$ANY_FAILED"` works correctly.

