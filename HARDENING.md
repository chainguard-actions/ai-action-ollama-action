<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA commit digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten. Failing references: `ai-action/setup-ollama@v2` (used twice) and `actions/cache@v6`.

Locations:

- `action.yml:27`
- `action.yml:33`
- `action.yml:38`

### script-injection (severity: high)

Sub-rule (b): In the 'Run model' step, the shell variable `$MODEL` (sourced from `inputs.model`) is expanded unquoted in the command `ollama run $MODEL "$(printf '%q' $PROMPT)"`. An attacker-controlled model name containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) can break out of the intended command and execute arbitrary shell commands. Similarly, `$PROMPT` is unquoted inside the command substitution `$(printf '%q' $PROMPT)`. Both variables must be double-quoted: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`. The env vars are set from `inputs.model` and `inputs.prompt` which are fully attacker-controlled.

Locations:

- `action.yml:49`

### github-env-injection (severity: high)

In the 'Run model' step, the `run:` block writes to `$GITHUB_OUTPUT` using a heredoc pattern. The env vars `MODEL` and `PROMPT` are sourced from `inputs.model` and `inputs.prompt` (untrusted attacker-controlled inputs). Neither value is sanitized with `printf '%s' ... | tr -d '\n\r'` before being used in the write to `$GITHUB_OUTPUT`. A newline embedded in `inputs.model` or `inputs.prompt` could inject additional key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting other step outputs. The randomized EOF delimiter only protects against heredoc terminator injection, not against newline injection in the values themselves.

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in action.yml: (1) Pinned ai-action/setup-ollama@v2 (used twice) to SHA 7c4a03fda24c7b1d7dbff1cf1c7b139c343fead9 and actions/cache@v6 to SHA 2c8a9bd7457de244a408f35966fab2fb45fda9c8, preserving original tags as comments. (2) Fixed script-injection by double-quoting $MODEL and $PROMPT in the shell command (now using $safe_model and $safe_prompt variables). (3) Fixed github-env-injection by sanitizing both MODEL and PROMPT with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT, preventing newline injection of additional key=value pairs.

