<!-- markdownlint-disable -->

# Hardening Report: pipelab-org--pipelock/v2.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pipelab-org--pipelock/v2.7.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Download Pipelock' writes values from workflow-controlled sources to special env files without the required `printf '%s' ... | tr -d '\n\r'` sanitization. VERSION comes from inputs.version (via PIPELOCK_VERSION) or the API response, and only has its leading 'v' stripped before `echo "version=${VERSION}" >> "$GITHUB_OUTPUT"`. INSTALL_DIR is built from the inherited RUNNER_TEMP env var and written with `echo "$INSTALL_DIR" >> "$GITHUB_PATH"`. Both values need sanitizing before they are written.

Locations:

- `action.yml:175`
- `action.yml:176`

### github-env-injection (severity: high)

Step 'Run audit' writes the unsanitized input-derived value CONFIG (from inputs.config via PIPELOCK_CONFIG) with `echo "config_path=${CONFIG}" >> "$GITHUB_OUTPUT"`. If the value contains a newline, an attacker can add extra outputs (for example audit_failed or critical_count) and get around the fail-on-findings gate. It also writes REPORT_DIR, which comes from the inherited RUNNER_TEMP, with `echo "config_path=${REPORT_DIR}/suggested.yaml" >> "$GITHUB_OUTPUT"` and no sanitization. Apply `printf '%s' "$VAR" | tr -d '\n\r'` before each write.

Locations:

- `action.yml:272`
- `action.yml:276`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

action.yml: in 'Download Pipelock', INSTALL_DIR and VERSION go through printf '%s' | tr -d '\n\r' before being written to GITHUB_PATH and GITHUB_OUTPUT. In 'Run audit', both config_path writes (the user CONFIG and REPORT_DIR/suggested.yaml) are sanitized the same way. Nothing else changed.

