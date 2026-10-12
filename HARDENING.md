<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three `uses:` references in action.yml use mutable tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tags are moved or overwritten:
- `ai-action/setup-ollama@v2` (line 27)
- `ai-action/setup-ollama@v2` (line 33)
- `actions/cache@v6` (line 40)

Locations:

- `action.yml:27`
- `action.yml:33`
- `action.yml:40`

### script-injection (severity: high)

Rule (b) violation: In the 'Run model' step, the shell variable `$MODEL` (sourced from `inputs.model`, an attacker-controlled input) is used **unquoted** in the run command: `ollama run $MODEL "$(printf '%q' $PROMPT)"`. An unquoted shell variable allows the shell to parse metacharacters (`;`, `|`, `&`, whitespace, globs, etc.) out of the value, enabling command injection. It must be quoted as `"$MODEL"`.

Locations:

- `action.yml:55`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned ai-action/setup-ollama@v2 to SHA 44a3e55e2fe1af8d9150329933b8a9f324f4ca74 (both occurrences at lines 27 and 33). 2. Pinned actions/cache@v6 to SHA 55cc8345863c7cc4c66a329aec7e433d2d1c52a9 (line 40). 3. Fixed script injection by quoting $MODEL as "$MODEL" in the ollama run command (line 55) — the variable was already passed via env: but was unquoted in the shell, allowing word-splitting and metacharacter injection.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted $PROMPT variable in the 'Run model' step's printf command substitution. Changed `$(printf '%q' $PROMPT)` to `$(printf '%q' "$PROMPT")` to prevent word-splitting and glob expansion on the attacker-controlled input before it reaches printf.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Run model' step of action.yml. Added sanitization of the MODEL input by creating SAFE_MODEL with `printf '%s' "$MODEL" | tr -d '\n\r'` to strip embedded newlines before using it in the ollama run command. The LLM response is still written using the heredoc form (response<<$EOF ... $EOF with randomized delimiter) which is the correct approach for multi-line values per the security guidelines - the random delimiter prevents heredoc injection and newlines should not be stripped from heredoc-written values. The PROMPT input continues to be escaped via printf '%q' before being passed to ollama.

