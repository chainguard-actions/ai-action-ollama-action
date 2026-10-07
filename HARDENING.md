<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tags are moved or overwritten. Failing references: `ai-action/setup-ollama@v2` (lines 29 and 32), `actions/cache@v5` (line 38).

Locations:

- `action.yml:29`
- `action.yml:32`
- `action.yml:38`

### script-injection (severity: high)

Rule (b) violation — unquoted shell variable expansions of untrusted inputs in the 'Run model' step. (1) `ollama run $MODEL` — the env var `$MODEL` (sourced from `inputs.model`) is unquoted, allowing shell metacharacter injection (e.g. semicolons, pipes, backticks). (2) `$(printf '%q' $PROMPT)` — the env var `$PROMPT` (sourced from `inputs.prompt`) is unquoted inside the command substitution, also allowing metacharacter injection. Both variables must be double-quoted: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"` .

Locations:

- `action.yml:49`

### github-env-injection (severity: high)

The 'Run model' step writes to `$GITHUB_OUTPUT` using a block that includes the output of `ollama run $MODEL "$(printf '%q' $PROMPT)"`. The env vars `MODEL` and `PROMPT` are set from `inputs.model` and `inputs.prompt` respectively (attacker-controlled). The `printf '%q'` call performs shell-argument quoting, NOT the required newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`) mandated before writes to `$GITHUB_OUTPUT`. An attacker-controlled prompt containing newlines could inject additional key=value pairs or heredoc delimiters into `$GITHUB_OUTPUT`, potentially overwriting other outputs.

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml: (1) Pinned ai-action/setup-ollama@v2 to SHA 2bdce2f0507c7a0765f8ba9ea6204cc8c9486b42 (both occurrences at lines 29 and 32) and actions/cache@v5 to SHA caa296126883cff596d87d8935842f9db880ef25 (line 38). (2) Double-quoted $MODEL and $PROMPT in the 'Run model' shell script to prevent metacharacter injection. (3) Fixed github-env-injection by capturing ollama output into 'raw', then sanitizing with 'printf "%s" "$raw" | tr -d "\n\r"' into 'safe' before writing to $GITHUB_OUTPUT, using the two-step pattern to avoid swallowing errors under errexit.

