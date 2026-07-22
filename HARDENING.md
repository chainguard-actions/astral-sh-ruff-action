<!-- markdownlint-disable -->

# Hardening Report: astral-sh--ruff-action/v4.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--ruff-action/v4.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a run: shell command string. In update-known-checksums.yml, `${{ secrets.GITHUB_TOKEN }}` is passed as a command-line argument directly in the run: block: `src/download/checksum/known-checksums.ts ${{ secrets.GITHUB_TOKEN }}`. This flows through YAML template substitution before the shell processes it, enabling injection if the value contains shell metacharacters.

Locations:

- `.github/workflows/update-known-checksums.yml:25`

### script-injection (severity: high)

Sub-rule (a): ${{ runner.temp }} expressions are directly interpolated inside a run: shell command string in the test-custom-manifest-file job. The offending lines are: `mkdir -p "${{ runner.temp }}/ruff-manifest-test"` and `printf 'print("hello")\n' > "${{ runner.temp }}/ruff-manifest-test/hello.py"`. Any ${{ ... }} in a run: block is a script-injection risk as it is substituted by the YAML template engine before the shell sees it.

Locations:

- `.github/workflows/test.yml:449`

### script-injection (severity: high)

Sub-rule (a): ${{ }} expressions are directly interpolated inside a run: shell command string in the all-tests-passed job. The offending lines are: `echo "All jobs passed: ${{ !(contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) }}"` and `exit ${{ (contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) && 1 || 0 }}`. These expressions are substituted by the YAML template engine before the shell processes them.

Locations:

- `.github/workflows/test.yml:506`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three script-injection findings:
1. update-known-checksums.yml line 25: Moved `${{ secrets.GITHUB_TOKEN }}` from the run: command line into an `env:` block as `GITHUB_TOKEN`, then referenced it as `"$GITHUB_TOKEN"` in the shell script.
2. test.yml line 449 (test-custom-manifest-file job): Moved `${{ runner.temp }}` from the mkdir/printf run: commands into an `env:` block as `RUNNER_TEMP_DIR`, then referenced it as `$RUNNER_TEMP_DIR` in the shell script.
3. test.yml line 506 (all-tests-passed job): Moved both `${{ !(contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) }}` and `${{ (contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) && 1 || 0 }}` expressions into an `env:` block as `ALL_PASSED` and `ANY_FAILED` respectively, then referenced them as `$ALL_PASSED` and `"$ANY_FAILED"` in the shell script.

