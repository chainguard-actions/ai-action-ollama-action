<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tags are moved:
- `uses: ai-action/setup-ollama@v2` (line 27)
- `uses: ai-action/setup-ollama@v2` (line 33)
- `uses: actions/cache@v5` (line 40)
All three should be pinned to their full commit SHA (e.g. `actions/cache@<40-hex-sha> # v5`).

Locations:

- `action.yml:27`
- `action.yml:33`
- `action.yml:40`

### script-injection (severity: high)

Rule (b) violation in the 'Run model' step: env vars `$MODEL` (sourced from `inputs.model`) and `$PROMPT` (sourced from `inputs.prompt`) are expanded unquoted inside the `run:` shell script.

- `ollama run $MODEL` — `$MODEL` is completely unquoted, allowing an attacker-controlled model name containing shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, globs) to inject arbitrary shell commands.
- `"$(printf '%q' $PROMPT)"` — `$PROMPT` is unquoted inside the command substitution, allowing the same class of metacharacter injection before `printf` even runs.

Fix: quote all expansions — `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"` — and consider routing inputs through `env:` with full quoting.

Locations:

- `action.yml:52`

### github-env-injection (severity: high)

The 'Run model' step writes to `$GITHUB_OUTPUT` using a heredoc block, but the values flowing into that write are not sanitized with `printf '%s' ... | tr -d '\n\r'` before the write.

- `$MODEL` (from `inputs.model`) is used unquoted in the shell command; a value containing a newline could break the heredoc structure or inject additional `key=value` pairs into `$GITHUB_OUTPUT`.
- `$PROMPT` (from `inputs.prompt`) is similarly unsanitized and unquoted inside the command substitution.

The randomized EOF delimiter (`EOF_OLLAMA_$(openssl rand -hex 12)`) protects against delimiter collision but does not sanitize the *content* written. Each input-derived value must be sanitized with `tr -d '\n\r'` before being written to `$GITHUB_OUTPUT`.

Locations:

- `action.yml:48`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:
1. unpinned-uses: Pinned `ai-action/setup-ollama@v2` to SHA `5f3116a5cd85594f351c3f8600a5ac08e63bc285` (both occurrences at lines 27 and 33) and `actions/cache@v5` to SHA `caa296126883cff596d87d8935842f9db880ef25` (line 40), preserving original tags as comments.
2. script-injection: Added proper double-quoting around `$MODEL` and `$PROMPT` in all shell expansions — `ollama run "$safe_model" "$(printf '%q' "$safe_prompt")"` — preventing shell metacharacter injection.
3. github-env-injection: Added sanitization of both `MODEL` and `PROMPT` inputs using `printf '%s' "$VAR" | tr -d '\n\r'` before writing to `$GITHUB_OUTPUT`, preventing newline injection attacks that could inject additional key=value pairs.

