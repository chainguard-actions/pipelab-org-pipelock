<!-- markdownlint-disable -->

# Hardening Report: pipelab-org--pipelock/v2.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pipelab-org--pipelock/v2.5.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Download Pipelock' writes values derived from workflow-controlled input to special env files without sanitization. VERSION starts as PIPELOCK_VERSION (from inputs.version) and only goes through `${VERSION#v}`. It is then written with `echo "version=${VERSION}" >> "$GITHUB_OUTPUT"`. INSTALL_DIR is built from the inherited env var RUNNER_TEMP and written with `echo "$INSTALL_DIR" >> "$GITHUB_PATH"`. Neither write is preceded by `printf '%s' ... | tr -d '\n\r'`, so a newline in the value could inject extra outputs or PATH entries. (Note: a non-'latest' VERSION that contained a newline would probably break the download and checksum check first, but the rule still requires sanitization.)

Locations:

- `action.yml:66`

### github-env-injection (severity: high)

Step 'Run audit' writes `echo "config_path=${CONFIG}" >> "$GITHUB_OUTPUT"`, where CONFIG comes from PIPELOCK_CONFIG (inputs.config). It also writes `echo "config_path=${REPORT_DIR}/suggested.yaml" >> "$GITHUB_OUTPUT"`, where REPORT_DIR comes from the inherited RUNNER_TEMP. Neither write is sanitized with `printf '%s' ... | tr -d '\n\r'`, so a multi-line inputs.config could inject arbitrary step outputs such as audit_failed or critical_count. A later step reads those outputs to decide pass or fail.

Locations:

- `action.yml:176`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed both github-env-injection findings in action.yml. In the 'Download Pipelock' step, INSTALL_DIR and VERSION now have newlines and carriage returns removed (printf '%s' | tr -d '\n\r') before they are written to GITHUB_PATH and GITHUB_OUTPUT. In the 'Run audit' step, the same is done to CONFIG and to the generated suggested.yaml path before each one is written as the config_path output. Each sanitized value is stored in its own variable. The fix uses plain pipelines of printf and tr, not commands nested inside other substitutions, so errexit still behaves as before. Values without newlines come out unchanged, so normal runs behave exactly as before. Nothing has been run to test it.

