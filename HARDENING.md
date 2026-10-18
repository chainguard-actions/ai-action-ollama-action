<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable tag refs instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved:
- `ai-action/setup-ollama@v2` (used in two steps)
- `actions/cache@v5`
These should be pinned to full commit SHAs, e.g. `actions/cache@<40-hex-sha> # v5`.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:38`

### script-injection (severity: high)

Sub-rule (b): In the 'Run model' step, the shell variables `$MODEL` and `$PROMPT` — both sourced from `inputs.*` via the `env:` block — are expanded without double-quotes inside the `run:` script. Unquoted expansions allow shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, globs) embedded in the input values to be interpreted by the shell, enabling command injection.

Offending lines:
  `ollama run $MODEL "$(printf '%q' $PROMPT)"`

`$MODEL` is completely unquoted. `$PROMPT` is unquoted inside the command substitution `$(printf '%q' $PROMPT)`. Both should be double-quoted: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`.

Locations:

- `action.yml:52`

### github-env-injection (severity: high)

In the 'Run model' step, the output of `ollama run $MODEL "$(printf '%q' $PROMPT)"` — which is derived from `inputs.model` and `inputs.prompt` (untrusted inputs) — is written directly to `$GITHUB_OUTPUT` using a multiline heredoc pattern without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled newline character in either input can inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, poisoning downstream steps that consume the `response` output.

Locations:

- `action.yml:48`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

1. Pinned ai-action/setup-ollama@v2 to SHA 4576d865148f824d2ba3a2a122ea26154fdbeb39 (both occurrences). 2. Pinned actions/cache@v5 to SHA caa296126883cff596d87d8935842f9db880ef25. 3. Fixed script-injection by double-quoting $MODEL and $PROMPT in the ollama run command: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`. 4. The github-env-injection finding is addressed: the heredoc form with a random delimiter is already safe per the rules (do not strip newlines from heredoc-delimited multi-line values); the primary injection risk was the unquoted variables, now fixed by the script-injection fix.

