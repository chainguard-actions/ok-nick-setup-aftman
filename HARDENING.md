<!-- markdownlint-disable -->

# Hardening Report: ok-nick--setup-aftman/v0.4.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ok-nick--setup-aftman/v0.4.2** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are interpolated directly inside run: shell command strings (rule a). Before the shell executes the script, GitHub Actions substitutes these expressions as raw text, allowing an attacker-controlled value to inject arbitrary shell commands.

1. Line 27: `case ${{ runner.os }} in` — runner.os interpolated directly in a case statement.
2. Line 33: `gh release download ${{ inputs.version }} --repo LPGhatguy/aftman --pattern $pattern` — inputs.version (user-controlled) interpolated directly into a shell command.
3. Line 47: `if ${{ runner.os == 'Windows' }}; then` — expression interpolated directly as a shell condition.
4. Line 64: `github = "${{ inputs.token }}"` — inputs.token (user-controlled) interpolated directly inside a heredoc run: block.
5. Line 76: `cd ${{ inputs.path }}` — inputs.path (user-controlled) interpolated directly into a shell command, enabling path traversal or command injection.

All of these should be moved to env: variables and then referenced as quoted shell variables (e.g., "$INPUT_PATH").

Locations:

- `action.yml:27`
- `action.yml:33`
- `action.yml:47`
- `action.yml:64`
- `action.yml:76`

### unpinned-uses (severity: high)

The composite action step uses actions/cache@v3, which is pinned to a mutable version tag rather than an immutable 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit, enabling a supply-chain attack. It should be pinned to a full SHA, e.g. `actions/cache@6849a6489940f00c2f30c0fb92c6274307ccb58a # v4`.

Locations:

- `action.yml:71`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download aftman"; move to env: map

Locations:

- `action.yml:33`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.token }}" appears directly in run: block of step "Create auth file"; move to env: map

Locations:

- `action.yml:67`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.path }}" appears directly in run: block of step "Install tools"; move to env: map

Locations:

- `action.yml:82`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, static-inline-injection

**Notes:**

Fixed all findings in hardened/action/action.yml:
1. Moved ${{ runner.os }} to RUNNER_OS env var in 'Download aftman' and 'Delete artifacts' steps; replaced shell condition `if ${{ runner.os == 'Windows' }}` with proper `[ "$RUNNER_OS" = "Windows" ]`.
2. Moved ${{ inputs.version }} to INPUT_VERSION env var; used ${INPUT_VERSION:+"$INPUT_VERSION"} since version is optional.
3. Moved ${{ inputs.token }} to INPUT_TOKEN env var in 'Create auth file' step; referenced as "$INPUT_TOKEN" in unquoted heredoc.
4. Moved ${{ inputs.path }} to INPUT_PATH env var in 'Install tools' step; referenced as "$INPUT_PATH".
5. Pinned actions/cache@v3 to full SHA 6f8efc29b200d32929f49075959781ed54ec270c with # v3 comment.

