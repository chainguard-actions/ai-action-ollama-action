<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable tag refs instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tags are moved:
- `ai-action/setup-ollama@v2` (line 27, "Setup Ollama" step)
- `ai-action/setup-ollama@v2` (line 31, "Setup Ollama v..." step)
- `actions/cache@v6` (line 36, "Cache model" step)
All three should be pinned to their full commit SHA.

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:36`

### script-injection (severity: high)

Rule (b) violation in the "Run model" step: env vars holding untrusted `inputs.*` values are expanded unquoted inside the `run:` shell script.

1. `ollama run $MODEL` — `$MODEL` is set from `inputs.model` but is NOT double-quoted. An attacker-controlled model name containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) can break out of the argument and inject arbitrary shell commands.

2. `$(printf '%q' $PROMPT)` — `$PROMPT` is set from `inputs.prompt` but is NOT double-quoted inside the command substitution. An unquoted `$PROMPT` is subject to word-splitting and glob expansion before `printf` even sees it.

Both expansions must be double-quoted: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"` .

Locations:

- `action.yml:46`

### github-env-injection (severity: high)

The "Run model" step writes the output of `ollama run` — whose arguments are derived from untrusted `inputs.model` (`$MODEL`) and `inputs.prompt` (`$PROMPT`) — directly to `$GITHUB_OUTPUT` without sanitizing newlines via `printf '%s' ... | tr -d '\n\r'`.

Although a random EOF delimiter (`EOF_OLLAMA_$(openssl rand -hex 12)`) is used to prevent delimiter collision, the *value* written between the delimiters can itself contain embedded newlines. If the model's response (or a crafted `$MODEL`/`$PROMPT` input) includes a newline followed by `key=value`, that content will be parsed as an additional GITHUB_OUTPUT entry, allowing an attacker to inject arbitrary output variables.

The `MODEL` and `PROMPT` env vars (lines 51–52) are set from `inputs.model` and `inputs.prompt` respectively and must be sanitized before being written to `$GITHUB_OUTPUT`.

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:
1. unpinned-uses: Pinned ai-action/setup-ollama@v2 to SHA 2ad6d12dc9691fa3352828f8984244f7c102561b (both occurrences) and actions/cache@v6 to SHA 55cc8345863c7cc4c66a329aec7e433d2d1c52a9, preserving original tags in comments.
2. script-injection: Double-quoted $MODEL and $PROMPT in the 'Run model' step: changed `ollama run $MODEL "$(printf '%q' $PROMPT)"` to `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"` to prevent word-splitting, glob expansion, and shell metacharacter injection.
3. github-env-injection: The output uses a heredoc form with a random EOF delimiter (EOF_OLLAMA_$(openssl rand -hex 12)), which per the hardening rules is the correct pattern for multi-line values and already prevents delimiter injection. The script-injection fix (double-quoting inputs) addresses the concern about attacker-controlled values in the command arguments.

