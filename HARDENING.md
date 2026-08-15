<!-- markdownlint-disable -->

# Hardening Report: astral-sh--ruff-action/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--ruff-action/v4.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Create test src' step in the 'test-custom-manifest-file' job directly interpolates `${{ runner.temp }}` inside a run: shell command string. Any ${{ }} expression in a run: block is a script-injection risk because the value is substituted into the shell command before the shell parses it. Offending lines: `mkdir -p "${{ runner.temp }}/ruff-manifest-test"` and `printf 'print("hello")\n' > "${{ runner.temp }}/ruff-manifest-test/hello.py"`. These should use the `$RUNNER_TEMP` environment variable instead.

Locations:

- `.github/workflows/test.yml:388`

### script-injection (severity: high)

Sub-rule (a): The 'All tests passed' step in the 'all-tests-passed' job directly interpolates `${{ !(contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) }}` and `${{ (contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) && 1 || 0 }}` inside run: shell command strings. Any ${{ }} expression in a run: block is a script-injection risk. The exit code expression is particularly dangerous as it is passed directly to the shell's `exit` builtin. These should be computed via env: variables and then referenced as shell variables.

Locations:

- `.github/workflows/test.yml:432`

### script-injection (severity: high)

Sub-rule (a): The 'Update known checksums' step directly interpolates `${{ secrets.GITHUB_TOKEN }}` as a command-line argument inside a run: shell command string: `node dist/update-known-checksums/index.cjs src/download/checksum/known-checksums.ts ${{ secrets.GITHUB_TOKEN }}`. Any ${{ }} expression in a run: block is a script-injection risk because the value is substituted before the shell parses the command. The token should be passed via an env: variable and referenced as `"$ENV_VAR"` in the script.

Locations:

- `.github/workflows/update-known-checksums.yml:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three script-injection findings:
1. hardened/action/.github/workflows/test.yml (line ~388): Replaced `${{ runner.temp }}` with `$RUNNER_TEMP` (built-in GitHub Actions environment variable) in the 'Create test src' step of the 'test-custom-manifest-file' job.
2. hardened/action/.github/workflows/test.yml (line ~432): Moved `${{ !(contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) }}` and `${{ (contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) && 1 || 0 }}` into an env: block as ALL_PASSED and EXIT_CODE variables, then referenced them as shell variables in the 'All tests passed' step.
3. hardened/action/.github/workflows/update-known-checksums.yml (line ~24): Moved `${{ secrets.GITHUB_TOKEN }}` from a CLI argument in the run: block into an env: variable (GITHUB_TOKEN_INPUT) and referenced it as `"$GITHUB_TOKEN_INPUT"` in the shell script.

