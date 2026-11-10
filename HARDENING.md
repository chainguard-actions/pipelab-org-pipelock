<!-- markdownlint-disable -->

# Hardening Report: pipelab-org--pipelock/v3.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pipelab-org--pipelock/v3.8.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Download Pipelock' writes values derived from untrusted input to special environment files without the `printf '%s' ... | tr -d '\n\r'` sanitization step. `echo "version=${VERSION}" >> "$GITHUB_OUTPUT"` writes VERSION, which comes from inputs.version through PIPELOCK_VERSION (or from the GitHub API response). `echo "$INSTALL_DIR" >> "$GITHUB_PATH"` writes a path built from the inherited RUNNER_TEMP env var. A value containing a newline could inject extra output keys or PATH entries. Fix: run each value through `safe=$(printf '%s' "$VERSION" | tr -d '\n\r')` before the write.

Locations:

- `action.yml:176`
- `action.yml:177`

### github-env-injection (severity: high)

Step 'Run audit' writes `echo "config_path=${CONFIG}" >> "$GITHUB_OUTPUT"`. CONFIG comes straight from inputs.config through the PIPELOCK_CONFIG env var and is not sanitized. A config input containing a newline could inject arbitrary step outputs such as audit_failed or critical_count, which the later 'Check results' gate relies on. The later `config_path=${REPORT_DIR}/suggested.yaml` write also uses a path built from the inherited RUNNER_TEMP. Fix: set `safe=$(printf '%s' "$CONFIG" | tr -d '\n\r')` and write that value instead.

Locations:

- `action.yml:283`
- `action.yml:291`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

action.yml: In 'Download Pipelock', INSTALL_DIR (written to GITHUB_PATH) and VERSION (written to GITHUB_OUTPUT) now have newlines stripped before the write. In 'Run audit', config_path is sanitized in both places it is written: the user's CONFIG value and the REPORT_DIR-based suggested.yaml path.

