<!-- markdownlint-disable -->

# Hardening Report: pipelab-org--pipelock/v2.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pipelab-org--pipelock/v2.3.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Download Pipelock' step writes VERSION to $GITHUB_OUTPUT with `echo "version=${VERSION}" >> "$GITHUB_OUTPUT"`. When the input is not 'latest', VERSION comes from inputs.version (env PIPELOCK_VERSION). The same step also writes INSTALL_DIR, which is built from the inherited RUNNER_TEMP, with `echo "$INSTALL_DIR" >> "$GITHUB_PATH"`. Neither value goes through `printf '%s' ... | tr -d '\n\r'` before the write, so newlines in it could inject extra outputs or PATH entries.

Locations:

- `action.yml:173`

### github-env-injection (severity: high)

The 'Run audit' step writes CONFIG to $GITHUB_OUTPUT with `echo "config_path=${CONFIG}" >> "$GITHUB_OUTPUT"`. CONFIG comes from inputs.config (env PIPELOCK_CONFIG). The step also writes `config_path=${REPORT_DIR}/suggested.yaml`, where REPORT_DIR is built from the inherited RUNNER_TEMP. Neither value is sanitized with `printf '%s' ... | tr -d '\n\r'`, so a newline in the input could inject arbitrary step outputs such as audit_failed or critical_count.

Locations:

- `action.yml:265`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Download step: INSTALL_DIR and VERSION are now cleaned with printf '%s' | tr -d '\n\r' before they are written to GITHUB_PATH and GITHUB_OUTPUT. Run audit step: CONFIG and REPORT_DIR are cleaned the same way before config_path is written to GITHUB_OUTPUT. Nothing else in the action was changed.

