<!-- markdownlint-disable -->

# Hardening Report: pipelab-org--pipelock/v2.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pipelab-org--pipelock/v2.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Download Pipelock' step writes values the workflow controls to special files without removing newlines first. VERSION comes from PIPELOCK_VERSION (${{ inputs.version }}); only a leading 'v' is stripped. It is written with `echo "version=${VERSION}" >> "$GITHUB_OUTPUT"`. INSTALL_DIR is built from the inherited RUNNER_TEMP env var and written with `echo "$INSTALL_DIR" >> "$GITHUB_PATH"`. There is no `printf '%s' ... | tr -d '\n\r'` step before either write, so a version input containing a newline can add extra output lines. Strictly, the curl download fails if the version is bad, which limits real-world risk, but this check still requires the sanitization.

Locations:

- `action.yml:167`
- `action.yml:168`

### github-env-injection (severity: high)

The 'Run audit' step writes values the workflow controls to $GITHUB_OUTPUT without removing newlines first. CONFIG comes from PIPELOCK_CONFIG (${{ inputs.config }}) and is written with `echo "config_path=${CONFIG}" >> "$GITHUB_OUTPUT"`. A newline in the input could add extra outputs, such as a fake audit_failed or critical_count value. REPORT_DIR, which comes from the inherited RUNNER_TEMP env var, is also written with `echo "config_path=${REPORT_DIR}/suggested.yaml" >> "$GITHUB_OUTPUT"` without sanitization.

Locations:

- `action.yml:263`
- `action.yml:267`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Both github-env-injection findings are fixed in action.yml. I haven't run the action or any tests to check the change.

- **'Download Pipelock' step:** VERSION and INSTALL_DIR now have newlines and carriage returns removed (`printf '%s' | tr -d '\n\r'`) before they're written to GITHUB_OUTPUT and GITHUB_PATH.
- **'Run audit' step:** CONFIG and REPORT_DIR get the same treatment before they're written as `config_path` to GITHUB_OUTPUT.

The cleaning happens on plain variables that are already set, not inside nested command substitutions, so the step still stops on errors as it did before. Nothing else in the action changed.

