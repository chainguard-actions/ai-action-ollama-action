<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml are pinned to mutable version tags instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tags are moved or overwritten:
- `ai-action/setup-ollama@v2` (line 29)
- `ai-action/setup-ollama@v2` (line 32)
- `actions/cache@v6` (line 37)
Each should be pinned to a full SHA, e.g. `actions/cache@<40-hex-sha> # v6`.

Locations:

- `action.yml:29`
- `action.yml:32`
- `action.yml:37`

### script-injection (severity: high)

Sub-rule (b): In the 'Run model' step (line 44), two env vars sourced from attacker-controlled inputs are expanded unquoted in the shell command:
1. `ollama run $MODEL` — `$MODEL` (from `inputs.model`) is completely unquoted, allowing shell metacharacter injection (`;`, `|`, `&`, `$(...)`, etc.).
2. `$(printf '%q' $PROMPT)` — `$PROMPT` (from `inputs.prompt`) is unquoted inside the command substitution, also allowing metacharacter injection before `printf` can quote it.
Both should be double-quoted: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`.

Locations:

- `action.yml:49`

### github-env-injection (severity: high)

In the 'Run model' step, the output of `ollama run $MODEL ...` (where `$MODEL` derives from `inputs.model` and `$PROMPT` derives from `inputs.prompt`) is written directly to `$GITHUB_OUTPUT` via `} >> $GITHUB_OUTPUT` without applying the required sanitization step (`printf '%s' ... | tr -d '\n\r'`) to strip embedded newlines. An attacker-controlled model name or prompt containing newline characters could inject additional key=value pairs or heredoc delimiters into `$GITHUB_OUTPUT`, potentially poisoning downstream step outputs.

Locations:

- `action.yml:51`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

1. Pinned ai-action/setup-ollama@v2 to SHA 4576d865148f824d2ba3a2a122ea26154fdbeb39 (both occurrences at lines 29 and 32). 2. Pinned actions/cache@v6 to SHA 55cc8345863c7cc4c66a329aec7e433d2d1c52a9 (line 37). 3. Fixed script-injection by double-quoting $MODEL and $PROMPT in the 'Run model' step: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`. 4. The github-env-injection finding is addressed: the output is written via a heredoc with a cryptographically random delimiter (EOF_OLLAMA_$(openssl rand -hex 12)), which per the hardening rules is the correct multi-line value form that already prevents newline injection — no tr sanitization is needed or appropriate for heredoc-delimited values. The script-injection fix also removes the unquoted variable expansion that was the primary injection vector.

