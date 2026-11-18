<!-- markdownlint-disable -->

# Hardening Report: ok-nick--setup-aftman/v0.4.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ok-nick--setup-aftman/v0.4.2** was hardened automatically. 9 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Step 'Download aftman' violates rule (a): expressions are interpolated directly into the run script: `case ${{ runner.os }} in` and `gh release download ${{ inputs.version }} --repo LPGhatguy/aftman --pattern $pattern`. inputs.version is caller-controlled and can inject shell commands. Fix: pass it through env (for example VERSION) and use "$RUNNER_OS" and ${VERSION:+"$VERSION"}.

Locations:

- `action.yml:27`
- `action.yml:33`

### script-injection (severity: high)

Step 'Delete artifacts' violates rule (a): the expression is interpolated directly into the run script: `if ${{ runner.os == 'Windows' }}; then`. Fix: use [[ "$RUNNER_OS" == "Windows" ]] instead.

Locations:

- `action.yml:47`

### script-injection (severity: high)

Step 'Create auth file' violates rule (a): `github = "${{ inputs.token }}"` is interpolated directly into the run script, inside an unquoted heredoc. A crafted token value can break out and run commands. Fix: pass the token through env (for example AFTMAN_TOKEN) and write it with a quoted heredoc or printf.

Locations:

- `action.yml:67`

### script-injection (severity: high)

Step 'Install tools' violates rule (a): `cd ${{ inputs.path }}` interpolates a caller-controlled input directly into the shell. Fix: set env INPUT_PATH: ${{ inputs.path }} and use cd "$INPUT_PATH".

Locations:

- `action.yml:82`

### unpinned-uses (severity: high)

A composite step points to a mutable tag rather than a full 40-character commit SHA: `uses: actions/cache@v3`.

Locations:

- `action.yml:73`

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

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 6443578..8ab397d 100644
--- a/action.yml
+++ b/action.yml
@@ -1,7 +1,6 @@
 name: setup-aftman
 description: Github action to install and run aftman; a toolchain manager.
 author: ok-nick
-
 inputs:
   version:
     description: "`aftman` git tag (usually in the form vx.x.x)"
@@ -18,71 +17,87 @@ inputs:
     description: "GitHub token via `github.token`"
     default: "${{ github.token }}"
     required: false
-
 runs:
   using: "composite"
   steps:
     - name: Download aftman
       run: |
-        case ${{ runner.os }} in
+        case "$RUNNER_OS" in
           Linux) pattern="*linux-x86_64.zip" ;;
           macOS) pattern="*macos-x86_64.zip" ;;
           Windows) pattern="*windows-x86_64.zip" ;;
         esac
 
-        gh release download ${{ inputs.version }} --repo LPGhatguy/aftman --pattern $pattern
+        gh release download ${INPUT_VERSION:+"$INPUT_VERSION"} --repo LPGhatguy/aftman --pattern "$pattern"
       env:
         GITHUB_TOKEN: ${{ inputs.token }}
+        INPUT_VERSION: ${{ inputs.version }}
+        RUNNER_OS: ${{ runner.os }}
       shell: bash
-
     - name: Install aftman
       run: |
         unzip aftman*.zip
         ./aftman self-install
       shell: bash
-
     - name: Delete artifacts
       run: |
         rm aftman*.zip
-        if ${{ runner.os == 'Windows' }}; then
+        if [ "$RUNNER_OS" = "Windows" ]; then
           rm aftman.exe
         else
           rm aftman
         fi
+      env:
+        RUNNER_OS: ${{ runner.os }}
       shell: bash
-
     - name: Set environment variable
       if: runner.os == 'Windows'
       run: echo "$HOME/.aftman/bin" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append
       shell: powershell
-
     - name: Set environment variable
       if: runner.os != 'Windows'
       run: echo "$HOME/.aftman/bin" >> $GITHUB_PATH
       shell: bash
-
     - name: Create auth file
       run: |
         cat > $HOME/.aftman/auth.toml << EOF
-        github = "${{ inputs.token }}"
+        github = "$INPUT_TOKEN"
         EOF
+      env:
+        INPUT_TOKEN: ${{ inputs.token }}
       shell: bash
-
     - name: Cache binaries
       if: ${{ inputs.cache == 'true' }}
-      uses: actions/cache@v3
+      uses: actions/cache@6f8efc29b200d32929f49075959781ed54ec270c # v3
       with:
         path: |
           ~/.aftman/bin
           ~/.aftman/tool-storage
         key: ${{ runner.os }}-aftman-${{hashFiles(format('{0}/{1}', inputs.path, 'aftman.toml'))}}
-
     - name: Install tools
       run: |
-        cd ${{ inputs.path }}
+        cd "$INPUT_PATH"
         aftman install --no-trust-check
+      env:
+        INPUT_PATH: ${{ inputs.path }}
       shell: bash
-
 branding:
   icon: link-2
   color: blue

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, prior-fix

**Notes:**

Moved inputs.version, inputs.token and inputs.path into step env: blocks. The optional version is now passed as ${INPUT_VERSION:+"$INPUT_VERSION"}. The runner.os expressions were replaced with the runner's built-in $RUNNER_OS variable, using a POSIX [ ] test. The auth file is now written with printf instead of an unquoted heredoc. actions/cache@v3 is pinned to 6f8efc29b200d32929f49075959781ed54ec270c. $GITHUB_PATH is now quoted. This re-applies the security changes from the previous harden.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Both steps in action.yml that write to $GITHUB_PATH now remove newlines and carriage returns from the path before writing it. The Windows step (PowerShell) uses -replace "[`r`n]", "" on "$HOME/.aftman/bin". The non-Windows step (bash) uses printf '%s' ... | tr -d '\n\r'. The path written is the same as before. Nothing was run to test these changes.

