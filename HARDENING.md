<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml are pinned to mutable tags rather than full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved or overwritten:
- `uses: ai-action/setup-ollama@v2` (Setup Ollama step)
- `uses: ai-action/setup-ollama@v2` (Setup Ollama v${{ inputs.version }} step)
- `uses: actions/cache@v5` (Cache model step)
All three should be pinned to a full SHA digest, e.g. `uses: actions/cache@<40-char-sha> # v5`.

Locations:

- `action.yml:25`
- `action.yml:30`
- `action.yml:36`

### script-injection (severity: high)

Sub-rule (b) violation in the 'Run model' step: the env vars `$MODEL` and `$PROMPT` — sourced from `inputs.model` and `inputs.prompt` via the `env:` block — are expanded unquoted inside the `run:` shell script.

Offending lines:
  `ollama run $MODEL "$(printf '%q' $PROMPT)"`

`$MODEL` is completely unquoted, allowing an attacker-controlled model name containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) to be interpreted by the shell. `$PROMPT` is also unquoted inside the command substitution `$(printf '%q' $PROMPT)`, which still allows shell metacharacter splitting before `printf` receives it.

Fix: quote both variables — `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"` — so the shell treats their values as single words.

Locations:

- `action.yml:50`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Pinned all three unpinned `uses:` references to full commit SHAs: `ai-action/setup-ollama@v2` → `@80ec1fde1ecef3f72b57cc8942bc7d6ff15e56cb # v2` (both occurrences) and `actions/cache@v5` → `@caa296126883cff596d87d8935842f9db880ef25 # v5`. Fixed script injection in the 'Run model' step by double-quoting both `$MODEL` and `$PROMPT` in the shell command: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`.

