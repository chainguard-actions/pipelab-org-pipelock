<!-- markdownlint-disable -->

# Hardening Report: pipelab-org--pipelock/v2.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pipelab-org--pipelock/v2.8.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Download Pipelock' step writes VERSION to $GITHUB_OUTPUT with `echo "version=${VERSION}" >> "$GITHUB_OUTPUT"`, and VERSION comes from inputs.version (env PIPELOCK_VERSION). No `printf '%s' ... | tr -d '\n\r'` step runs first, so a newline in the input can inject extra outputs. The same step also writes INSTALL_DIR, which is built from the inherited RUNNER_TEMP, to $GITHUB_PATH without sanitizing it.

Locations:

- `action.yml:172`
- `action.yml:171`

### github-env-injection (severity: high)

The 'Run audit' step writes CONFIG to $GITHUB_OUTPUT with `echo "config_path=${CONFIG}" >> "$GITHUB_OUTPUT"`, and CONFIG comes from inputs.config (env PIPELOCK_CONFIG). No `printf '%s' ... | tr -d '\n\r'` step runs first, so a newline in the input can inject arbitrary step outputs. Later steps use that output as CONFIG_PATH.

Locations:

- `action.yml:276`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

action.yml: the Download step now removes \n and \r from INSTALL_DIR (written to GITHUB_PATH) and VERSION (written to GITHUB_OUTPUT), using printf '%s' | tr -d '\n\r'. The Run audit step does the same for CONFIG and for the REPORT_DIR-derived suggested.yaml path before writing config_path. No other behavior changed.

