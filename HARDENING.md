<!-- markdownlint-disable -->

# Hardening Report: astral-sh--ruff-action--/v4.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **astral-sh--ruff-action--/v4.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: `${{ runner.temp }}` is directly interpolated inside a `run:` shell command string in the `test-custom-manifest-file` job. Any `${{ ... }}` expression directly inside a `run:` block is a script-injection finding. The offending lines are: `mkdir -p "${{ runner.temp }}/ruff-manifest-test"` and `printf 'print("hello")\n' > "${{ runner.temp }}/ruff-manifest-test/hello.py"`.

Locations:

- `.github/workflows/test.yml:432`

### script-injection (severity: high)

Rule (a) violation: `${{ !(contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) }}` and `exit ${{ (contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) && 1 || 0 }}` are directly interpolated inside a `run:` shell command string in the `all-tests-passed` job. Any `${{ ... }}` expression directly inside a `run:` block is a script-injection finding.

Locations:

- `.github/workflows/test.yml:490`

### script-injection (severity: high)

Rule (a) violation: `${{ secrets.GITHUB_TOKEN }}` is directly interpolated inside a `run:` shell command string as a CLI argument to a node script: `node dist/update-known-checksums/index.cjs src/download/checksum/known-checksums.ts ${{ secrets.GITHUB_TOKEN }}`. Any `${{ ... }}` expression directly inside a `run:` block is a script-injection finding.

Locations:

- `.github/workflows/update-known-checksums.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three script-injection findings:
1. test.yml (test-custom-manifest-file job): Moved `${{ runner.temp }}` into an `env:` block as `RUNNER_TEMP` and replaced inline expressions with `$RUNNER_TEMP` in the shell script.
2. test.yml (all-tests-passed job): Moved `${{ !(contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) }}` and the exit-code expression into `env:` block as `ALL_PASSED` and `HAS_FAILURE`, referenced as plain env vars in the shell script.
3. update-known-checksums.yml: Moved `${{ secrets.GITHUB_TOKEN }}` from a CLI argument into an `env:` block as `GITHUB_TOKEN`, referenced as `"$GITHUB_TOKEN"` in the node command.

