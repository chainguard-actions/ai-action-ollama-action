<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable tag refs instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tags are moved or overwritten:
- `ai-action/setup-ollama@v2` (line 28)
- `ai-action/setup-ollama@v2` (line 31)
- `actions/cache@v6` (line 35)
These should be pinned to full commit SHAs, e.g. `actions/cache@<40-hex-sha> # v6`.

Locations:

- `action.yml:28`
- `action.yml:31`
- `action.yml:35`

### script-injection (severity: high)

Rule (b) violation: In the 'Run model' step, the env var `$MODEL` (sourced from `inputs.model`) is expanded unquoted in the shell command `ollama run $MODEL "$(printf '%q' $PROMPT)"` (line 47). An unquoted shell variable expansion allows an attacker-controlled `inputs.model` value containing shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, globs) to be interpreted by the shell, enabling command injection. Similarly, `$PROMPT` is unquoted inside the command substitution `$(printf '%q' $PROMPT)`. Both variables must be double-quoted: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`.

Locations:

- `action.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed three unpinned `uses:` references by resolving them to full commit SHAs: `ai-action/setup-ollama@v2` → `@bb1a15e4315698e668874dc3aad323d7c6d57ede # v2` (both occurrences) and `actions/cache@v6` → `@55cc8345863c7cc4c66a329aec7e433d2d1c52a9 # v6`. Fixed script injection by double-quoting `$MODEL` and `$PROMPT` in the 'Run model' step shell command: changed `ollama run $MODEL "$(printf '%q' $PROMPT)"` to `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`. The env: block pattern was already correct; only the quoting in the run: script needed fixing.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in hardened/action/action.yml by adding sanitization of $MODEL and $PROMPT inputs before they are used in the 'Run model' step. Added two lines: `SAFE_MODEL=$(printf '%s' "$MODEL" | tr -d '\n\r')` and `SAFE_PROMPT=$(printf '%s' "$PROMPT" | tr -d '\n\r')`, then replaced $MODEL and $PROMPT with $SAFE_MODEL and $SAFE_PROMPT in the ollama command. This prevents newline injection attacks where attacker-controlled inputs containing embedded newlines could inject additional key=value pairs into $GITHUB_OUTPUT. The env: block correctly keeps the ${{ inputs.* }} expressions out of the run script (already correct), and the sanitization is applied at the shell level before the values are used.

