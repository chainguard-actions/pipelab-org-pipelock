<!-- markdownlint-disable -->

# Hardening Report: luckyPipewrench--pipelock/v3.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **luckyPipewrench--pipelock/v3.8.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Download Pipelock' step writes input-derived data to special environment files without sanitizing it. VERSION comes from inputs.version through PIPELOCK_VERSION, and `echo "version=${VERSION}" >> "$GITHUB_OUTPUT"` writes it with no `printf '%s' ... | tr -d '\n\r'` step. The same step also runs `echo "$INSTALL_DIR" >> "$GITHUB_PATH"`. INSTALL_DIR is built from the inherited RUNNER_TEMP env var and is also not sanitized.

Locations:

- `action.yml:183`
- `action.yml:182`

### github-env-injection (severity: high)

The 'Run audit' step writes the CONFIG variable to GITHUB_OUTPUT without stripping newlines: `echo "config_path=${CONFIG}" >> "$GITHUB_OUTPUT"`. CONFIG comes from inputs.config through PIPELOCK_CONFIG. A newline in the input could inject extra step outputs, such as audit_failed or critical_count. Sanitize it first, e.g. `safe=$(printf '%s' "$CONFIG" | tr -d '\n\r')`.

Locations:

- `action.yml:285`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In action.yml, the 'Download Pipelock' step now runs INSTALL_DIR and VERSION through `printf '%s' ... | tr -d '\n\r'` before writing them to GITHUB_PATH and GITHUB_OUTPUT. The 'Run audit' step does the same to CONFIG before it writes config_path to GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In action.yml's 'Run audit' step, REPORT_DIR now has \n and \r removed into SAFE_REPORT_DIR before it is written to config_path in $GITHUB_OUTPUT. This is the same fix the download step uses for INSTALL_DIR.

