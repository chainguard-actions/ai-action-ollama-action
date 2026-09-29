<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: In the 'Run model' step, the env var $MODEL (sourced from inputs.model) is used unquoted in the shell command `ollama run $MODEL "$(printf '%q' $PROMPT)"`. An unquoted shell variable expansion allows shell metacharacters in the input to be interpreted by the shell, enabling command injection. Additionally, $PROMPT is unquoted inside the command substitution `$(printf '%q' $PROMPT)`. Both variables should be double-quoted: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`.

Locations:

- `action.yml:47`

### github-env-injection (severity: high)

In the 'Run model' step, the output of `ollama run $MODEL ...` (which incorporates inputs.model and inputs.prompt via env vars MODEL and PROMPT) is written directly to $GITHUB_OUTPUT using `} >> $GITHUB_OUTPUT` without first sanitizing the values with `printf '%s' ... | tr -d '\n\r'`. A newline in inputs.model or inputs.prompt could inject additional key=value pairs into $GITHUB_OUTPUT, potentially overwriting other outputs or causing unexpected behavior.

Locations:

- `action.yml:44`

### unpinned-uses (severity: high)

Three `uses:` references use mutable tag refs instead of pinned full-length SHA commit hashes, making the action vulnerable to supply-chain attacks if those tags are moved:
- `ai-action/setup-ollama@v2` (used twice, in 'Setup Ollama' and 'Setup Ollama v...' steps)
- `actions/cache@v5` (used in 'Cache model' step)
All should be pinned to their full 40-character commit SHA.

Locations:

- `action.yml:28`
- `action.yml:33`
- `action.yml:39`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:
1. unpinned-uses: Pinned ai-action/setup-ollama@v2 to full SHA bb1a15e4315698e668874dc3aad323d7c6d57ede (used in two steps) and actions/cache@v5 to full SHA caa296126883cff596d87d8935842f9db880ef25.
2. script-injection: Double-quoted $MODEL and $PROMPT in the 'Run model' step shell command: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`. Also double-quoted $GITHUB_OUTPUT.
3. github-env-injection: Captured ollama output into $raw, sanitized with `printf '%s' "$raw" | tr -d '\n\r'` into $safe, and wrote only $safe to $GITHUB_OUTPUT to prevent newline injection attacks.

