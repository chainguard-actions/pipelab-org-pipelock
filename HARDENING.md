<!-- markdownlint-disable -->

# Hardening Report: pipelab-org--pipelock/v2.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pipelab-org--pipelock/v2.6.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Download Pipelock' step writes values the caller controls to special files without first stripping newlines with printf '%s' ... | tr -d '\n\r'. VERSION comes from inputs.version (env PIPELOCK_VERSION) and is written by `echo "version=${VERSION}" >> "$GITHUB_OUTPUT"`. Only a leading 'v' is removed, so a version containing a newline could add extra outputs. INSTALL_DIR is built from the inherited RUNNER_TEMP env var and is written by `echo "$INSTALL_DIR" >> "$GITHUB_PATH"`, also without sanitization.

Locations:

- `action.yml:169`
- `action.yml:168`

### github-env-injection (severity: high)

The 'Run audit' step writes CONFIG, which comes from inputs.config (env PIPELOCK_CONFIG), by `echo "config_path=${CONFIG}" >> "$GITHUB_OUTPUT"` without the printf '%s' | tr -d '\n\r' step, so a config value containing a newline can inject extra step outputs. The fallback `echo "config_path=${REPORT_DIR}/suggested.yaml" >> "$GITHUB_OUTPUT"` also uses REPORT_DIR, which is built from the inherited RUNNER_TEMP, without sanitization.

Locations:

- `action.yml:262`
- `action.yml:266`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Both github-env-injection findings are fixed in action.yml. I haven't run the action or any tests after the change.

Each flagged value now goes through `printf '%s' ... | tr -d '\n\r'` before it is written:
- **Download Pipelock step:** `INSTALL_DIR` is cleaned before it goes to `GITHUB_PATH`, and `VERSION` before it goes to `GITHUB_OUTPUT`.
- **Run audit step:** `CONFIG` and `REPORT_DIR` are cleaned before they are written as `config_path` to `GITHUB_OUTPUT`.

The cleaning only wraps plain variables, never a command, so a failing command still stops the step as it did before.

