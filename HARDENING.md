<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml are pinned to mutable tags instead of full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved:
- `ai-action/setup-ollama@v2` (used twice, lines 27 and 31)
- `actions/cache@v5` (line 36)
These should each be pinned to a full SHA digest, e.g. `actions/cache@<40-hex-sha> # v5`.

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:36`

### script-injection (severity: high)

Sub-rule (b) violation: In the 'Run model' step, the env var `$MODEL` (sourced from `inputs.model`, which is attacker-controlled) is expanded **unquoted** inside the `run:` shell command:

```
ollama run $MODEL "$(printf '%q' $PROMPT)"
```

An attacker supplying a model name containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) can cause arbitrary command execution. The fix is to double-quote the expansion: `ollama run "$MODEL" ...`.

Note: `$PROMPT` is passed through `printf '%q'` and then double-quoted, which is safer, but `$MODEL` has no such protection.

Locations:

- `action.yml:48`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned ai-action/setup-ollama@v2 (both occurrences) to SHA 7251aeb8bd484f01af7bff1f887073bcbd82df5c # v2. 2. Pinned actions/cache@v5 to SHA caa296126883cff596d87d8935842f9db880ef25 # v5. 3. Fixed script injection by double-quoting $MODEL in the 'ollama run' command to prevent shell metacharacter injection from attacker-controlled model names.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted $PROMPT variable in the 'Run model' step of action.yml. Changed `$(printf '%q' $PROMPT)` to `$(printf '%q' "$PROMPT")` to prevent shell word-splitting and glob expansion of the attacker-controlled prompt input before it reaches printf.

