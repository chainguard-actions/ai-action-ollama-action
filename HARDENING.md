<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml are pinned to mutable tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved:
- `uses: ai-action/setup-ollama@v2` (appears twice, lines 27 and 31)
- `uses: actions/cache@v6` (line 35)
All three should be pinned to a full SHA, e.g. `uses: actions/checkout@<40-hex-sha> # v6`.

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:35`

### script-injection (severity: high)

Rule (b) violation — unquoted shell variable expansions of untrusted inputs in the 'Run model' step (line ~46):
1. `ollama run $MODEL` — `$MODEL` is unquoted. It is set from `inputs.model` via the `env:` block. An attacker-controlled model name containing shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, globs) will be parsed by the shell before execution.
2. `$(printf '%q' $PROMPT)` — `$PROMPT` is unquoted inside the command substitution. It is set from `inputs.prompt` via the `env:` block. The unquoted expansion allows word-splitting and glob expansion before `printf` even receives the value.
Fix: quote both variables — `"$MODEL"` and `"$PROMPT"`.

Locations:

- `action.yml:46`

### github-env-injection (severity: high)

The 'Run model' step writes the output of `ollama run $MODEL "$(printf '%q' $PROMPT)"` — which incorporates the untrusted `inputs.model` (via `$MODEL`) and `inputs.prompt` (via `$PROMPT`) — directly to `$GITHUB_OUTPUT` using a heredoc-style block (`} >> $GITHUB_OUTPUT`) without first sanitizing the values with `printf '%s' "$VAR" | tr -d '\n\r'`. A value containing embedded newlines could inject arbitrary key=value pairs or heredoc delimiters into `$GITHUB_OUTPUT`, potentially overwriting other outputs or poisoning the environment of downstream steps.
Fix: sanitize both `$MODEL` and `$PROMPT` before writing to `$GITHUB_OUTPUT`, e.g.:
  safe_model=$(printf '%s' "$MODEL" | tr -d '\n\r')
  safe_prompt=$(printf '%s' "$PROMPT" | tr -d '\n\r')

Locations:

- `action.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:
1. unpinned-uses: Pinned `ai-action/setup-ollama@v2` (both occurrences, lines 27 and 31) to SHA `6782d243db5fbcbf7583532ce0b5122f64242689 # v2`, and `actions/cache@v6` (line 35) to SHA `55cc8345863c7cc4c66a329aec7e433d2d1c52a9 # v6`.
2. script-injection: Quoted `$MODEL` → `"$safe_model"` and `$PROMPT` → `"$safe_prompt"` (and `"$PROMPT"` inside the printf call) in the 'Run model' step.
3. github-env-injection: Added sanitization of MODEL and PROMPT via `printf '%s' "$VAR" | tr -d '\n\r'` into `safe_model` and `safe_prompt` variables before they are used in the heredoc written to `$GITHUB_OUTPUT`. Also quoted `$GITHUB_OUTPUT` and `$EOF` for robustness.

