<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable tags instead of full 40-character commit SHA digests, making the action vulnerable to supply-chain attacks if the referenced tags are moved:
- `ai-action/setup-ollama@v2` (used twice, lines 27 and 31)
- `actions/cache@v5` (line 36)
All three should be pinned to their full SHA digest (e.g. `actions/cache@<40-hex-sha> # v5`).

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:36`

### script-injection (severity: high)

Sub-rule (b): In the 'Run model' step, the shell variables `$MODEL` and `$PROMPT` — both sourced from `inputs.*` via the `env:` block — are expanded without double-quoting inside the `run:` script. Specifically:
  `ollama run $MODEL "$(printf '%q' $PROMPT)"`
`$MODEL` is completely unquoted, allowing shell metacharacter injection (`;`, `|`, `&`, etc.) from the `inputs.model` value. `$PROMPT` is also unquoted inside the command substitution `$(printf '%q' $PROMPT)`. These should be `"$MODEL"` and `"$PROMPT"` respectively.

Locations:

- `action.yml:46`

### github-env-injection (severity: high)

In the 'Run model' step, the values of `inputs.model` and `inputs.prompt` are passed via env vars `$MODEL` and `$PROMPT` and then written to `$GITHUB_OUTPUT` using a heredoc-style multiline write (`} >> $GITHUB_OUTPUT`) without any sanitization. The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is never applied before the write. A newline embedded in `inputs.model` or `inputs.prompt` could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially poisoning downstream steps.

Locations:

- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:
1. unpinned-uses: Pinned ai-action/setup-ollama@v2 (both occurrences, lines 27 and 31) to SHA 5f3116a5cd85594f351c3f8600a5ac08e63bc285, and actions/cache@v5 (line 36) to SHA caa296126883cff596d87d8935842f9db880ef25. Tag comments preserved for readability.
2. script-injection: Replaced unquoted $MODEL and $PROMPT with properly double-quoted "$safe_model" and "$safe_prompt" (and "$safe_prompt" inside the printf command substitution).
3. github-env-injection: Added sanitization of MODEL and PROMPT inputs using `printf '%s' "$VAR" | tr -d '\n\r'` before use, storing results in safe_model and safe_prompt variables. These sanitized values are used in the ollama run command, preventing newline injection into $GITHUB_OUTPUT.

