<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three `uses:` references in action.yml use mutable tag refs instead of pinned 40-character SHA commit hashes. If these tags are moved or compromised, the action will silently execute different code. Failing references: `ai-action/setup-ollama@v2` (used twice) and `actions/cache@v5`.

Locations:

- `action.yml:24`
- `action.yml:29`
- `action.yml:35`

### script-injection (severity: high)

Rule (b) violation: In the 'Run model' step, the env vars `$MODEL` and `$PROMPT` — sourced from `inputs.model` and `inputs.prompt` respectively — are expanded unquoted inside the `run:` shell script. The command `ollama run $MODEL "$(printf '%q' $PROMPT)"` leaves `$MODEL` completely unquoted, allowing an attacker-supplied model name containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) to inject arbitrary shell commands. `$PROMPT` is also unquoted inside the command substitution `$(printf '%q' $PROMPT)`, which is similarly unsafe.

Locations:

- `action.yml:47`

### github-env-injection (severity: high)

The 'Run model' step writes the output of `ollama run $MODEL ...` directly to `$GITHUB_OUTPUT` using a heredoc append without sanitizing newlines. The values of `$MODEL` (from `inputs.model`) and `$PROMPT` (from `inputs.prompt`) are attacker-controlled. An attacker could embed newline characters in these inputs to inject additional key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting other outputs consumed by downstream steps. The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent before the write.

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in action.yml:
1. unpinned-uses: Pinned ai-action/setup-ollama@v2 to SHA 2bdce2f0507c7a0765f8ba9ea6204cc8c9486b42 (both occurrences) and actions/cache@v5 to SHA caa296126883cff596d87d8935842f9db880ef25, with original tags preserved as comments.
2. script-injection: Added double-quotes around $MODEL and $PROMPT in the shell script to prevent shell metacharacter injection.
3. github-env-injection: Captured ollama output into 'raw', then sanitized with 'printf | tr -d \n\r' into 'safe' before writing to $GITHUB_OUTPUT to prevent newline injection attacks.

