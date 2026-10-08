<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml pin to mutable version tags instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved:
- `ai-action/setup-ollama@v2` (used twice, lines 28 and 31)
- `actions/cache@v6` (line 36)
These should be pinned to full SHA digests, e.g. `actions/cache@<40-hex-sha> # v6`.

Locations:

- `action.yml:28`
- `action.yml:31`
- `action.yml:36`

### script-injection (severity: high)

Rule (b) violation in the 'Run model' step: the shell variables `$MODEL` and `$PROMPT` — which hold values sourced from `inputs.model` and `inputs.prompt` via the `env:` block — are expanded **unquoted** inside the `run:` script.

- `ollama run $MODEL` — `$MODEL` is unquoted; an attacker-controlled model name containing shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, globs) will be parsed by the shell before being passed to `ollama`.
- `$(printf '%q' $PROMPT)` — `$PROMPT` is unquoted inside the command substitution; shell word-splitting and glob expansion apply to the value before `printf` ever sees it.

Both variables must be double-quoted: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"` (or the prompt passed via a safe mechanism).

Locations:

- `action.yml:48`

### github-env-injection (severity: high)

The 'Run model' step writes to `$GITHUB_OUTPUT` using a heredoc-style multiline delimiter block, where the response body is derived from `ollama run $MODEL` whose output is influenced by `inputs.model` and `inputs.prompt` (via `$MODEL`/`$PROMPT`). Neither `$MODEL` nor `$PROMPT` is sanitized with `printf '%s' ... | tr -d '\n\r'` before the write. A newline embedded in `inputs.prompt` or `inputs.model` could inject additional `key=value` pairs into `$GITHUB_OUTPUT`, allowing a caller to set arbitrary step outputs. The write at line 50 (`} >> $GITHUB_OUTPUT`) must be preceded by sanitization of all untrusted inputs that influence the written content.

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:
1. unpinned-uses: Pinned `ai-action/setup-ollama@v2` (both occurrences) to SHA `7251aeb8bd484f01af7bff1f887073bcbd82df5c # v2`, and `actions/cache@v6` to SHA `55cc8345863c7cc4c66a329aec7e433d2d1c52a9 # v6`.
2. script-injection: Added sanitized variables `safe_model` and `safe_prompt`, then double-quoted all variable expansions in the shell script (`"$safe_model"`, `"$safe_prompt"`).
3. github-env-injection: Sanitized `$MODEL` and `$PROMPT` with `printf '%s' "$VAR" | tr -d '\n\r'` before use in the heredoc block that writes to `$GITHUB_OUTPUT`, preventing newline injection. Also quoted `$GITHUB_OUTPUT` and `$EOF` for correctness.

