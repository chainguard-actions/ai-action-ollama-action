<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable version tags instead of pinned full-length SHA digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten:
- `ai-action/setup-ollama@v2` (line 30)
- `ai-action/setup-ollama@v2` (line 34)
- `actions/cache@v6` (line 40)
Each should be replaced with a full 40-character commit SHA (e.g. `uses: actions/cache@<sha> # v6`).

Locations:

- `action.yml:30`
- `action.yml:34`
- `action.yml:40`

### script-injection (severity: high)

Rule (b) violation in the 'Run model' step: the shell variable `$MODEL` (sourced from `inputs.model` via the `env:` block) is expanded **unquoted** in the `run:` script on line 52: `ollama run $MODEL "$(printf '%q' $PROMPT)"`. An unquoted shell variable expansion allows an attacker-controlled value containing shell metacharacters (spaces, semicolons, pipes, backticks, `$(...)`, etc.) to be parsed by the shell as separate tokens or commands, enabling command injection. The fix is to quote it: `ollama run "$MODEL" ...`.

Locations:

- `action.yml:52`

### github-env-injection (severity: high)

The 'Run model' step writes to `$GITHUB_OUTPUT` (line 54) using the output of `ollama run $MODEL "$(printf '%q' $PROMPT)"`, where both `$MODEL` (from `inputs.model`) and `$PROMPT` (from `inputs.prompt`) are attacker-controlled inputs passed via the `env:` block. Neither value is sanitized with `printf '%s' ... | tr -d '\n\r'` before the write. A newline embedded in `$MODEL` or `$PROMPT` could inject additional `key=value` pairs into `$GITHUB_OUTPUT`, allowing an attacker to override subsequent step outputs. The randomized EOF delimiter protects the multiline heredoc header, but the *content* of the response value is still unsanitized.

Locations:

- `action.yml:54`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

1. Pinned ai-action/setup-ollama@v2 (both occurrences, lines 30 and 34) to full SHA 80ec1fde1ecef3f72b57cc8942bc7d6ff15e56cb. 2. Pinned actions/cache@v6 (line 40) to full SHA 55cc8345863c7cc4c66a329aec7e433d2d1c52a9. 3. Fixed script injection by quoting $MODEL as "$MODEL" in the ollama run command. 4. Fixed github-env-injection by capturing the ollama output into a variable and sanitizing it with `tr -d '\n\r'` before writing to $GITHUB_OUTPUT, preventing newline-based output injection.

