<!-- markdownlint-disable -->

# Hardening Report: pipelab-org--pipelock/v2.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pipelab-org--pipelock/v2.4.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Download Pipelock' step writes VERSION to $GITHUB_OUTPUT without sanitizing it. VERSION comes from inputs.version (via PIPELOCK_VERSION) and is written as `echo "version=${VERSION}" >> "$GITHUB_OUTPUT"`. Nothing applies `printf '%s' ... | tr -d '\n\r'` first. The step also writes INSTALL_DIR to $GITHUB_PATH (`echo "$INSTALL_DIR" >> "$GITHUB_PATH"`). INSTALL_DIR is built from the inherited RUNNER_TEMP env var and is not sanitized either. A value containing a newline could inject extra outputs or paths.

Locations:

- `action.yml:171`
- `action.yml:170`

### github-env-injection (severity: high)

The 'Run audit' step writes CONFIG to $GITHUB_OUTPUT without sanitizing it. CONFIG comes from inputs.config (via PIPELOCK_CONFIG) and is written as `echo "config_path=${CONFIG}" >> "$GITHUB_OUTPUT"`. Nothing applies `printf '%s' ... | tr -d '\n\r'` first. A newline in the input would let an attacker inject arbitrary step outputs, such as audit_failed or critical_count, which the later 'Check results' step reads. The step also writes `config_path=${REPORT_DIR}/suggested.yaml`, where REPORT_DIR comes from the inherited RUNNER_TEMP, without sanitizing it.

Locations:

- `action.yml:268`
- `action.yml:271`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In action.yml, the 'Download Pipelock' step now runs INSTALL_DIR and VERSION through printf '%s' | tr -d '\n\r' before writing them to GITHUB_PATH and GITHUB_OUTPUT. The 'Run audit' step does the same to CONFIG and REPORT_DIR before writing config_path. Nothing else changed. No tests were run.

