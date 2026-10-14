<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable tag refs instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tags are moved or overwritten.
- `ai-action/setup-ollama@v2` (line 28)
- `ai-action/setup-ollama@v2` (line 33)
- `actions/cache@v5` (line 37)

Locations:

- `action.yml:28`
- `action.yml:33`
- `action.yml:37`

### script-injection (severity: high)

Sub-rule (b): In the 'Run model' step, the env var `$MODEL` (sourced from `inputs.model`, attacker-controlled) is expanded **unquoted** in the shell command `ollama run $MODEL "$(printf '%q' $PROMPT)"`. An attacker-supplied model name containing shell metacharacters (`;`, `|`, `&`, backticks, `$(...)`, etc.) would be interpreted by the shell, enabling command injection. Similarly, `$PROMPT` is unquoted inside the command substitution `$(printf '%q' $PROMPT)`, allowing word-splitting and glob expansion before `printf` even receives it. Both expansions must be double-quoted: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`.

Locations:

- `action.yml:50`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned all three unpinned `uses:` references to full 40-character SHA digests: `ai-action/setup-ollama@v2` → `@2ad6d12dc9691fa3352828f8984244f7c102561b # v2` (both occurrences at lines 28 and 33), and `actions/cache@v5` → `@caa296126883cff596d87d8935842f9db880ef25 # v5` (line 37). 2. Fixed script injection in the 'Run model' step by double-quoting both `$MODEL` and `$PROMPT` in the shell command: changed `ollama run $MODEL "$(printf '%q' $PROMPT)"` to `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`.

