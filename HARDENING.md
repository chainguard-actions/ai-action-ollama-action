<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable tag refs instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tags are moved or overwritten:
- `ai-action/setup-ollama@v2` (line 27)
- `ai-action/setup-ollama@v2` (line 32)
- `actions/cache@v6` (line 40)

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:40`

### script-injection (severity: high)

Sub-rule (b) violation in the 'Run model' step: the shell variables `$MODEL` and `$PROMPT` — sourced from untrusted inputs `inputs.model` and `inputs.prompt` respectively — are expanded without double-quoting inside the `run:` script. An unquoted `$MODEL` in `ollama run $MODEL ...` allows shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) embedded in the input to be interpreted by the shell, enabling command injection. Similarly, `$PROMPT` is unquoted inside the command substitution `$(printf '%q' $PROMPT)`. Both should be double-quoted: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`.

Locations:

- `action.yml:56`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed three unpinned `uses:` references by resolving their full commit SHAs: `ai-action/setup-ollama@v2` → SHA `e1b426c70229469e478466cb2a00e15eb2542ce1` (both occurrences), and `actions/cache@v6` → SHA `55cc8345863c7cc4c66a329aec7e433d2d1c52a9`. Fixed script injection by double-quoting `$MODEL` and `$PROMPT` in the shell script (`ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`) — the variables were already in the `env:` block but were unquoted in the shell, allowing shell metacharacters to be interpreted.

