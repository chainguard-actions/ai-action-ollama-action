<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tags are moved or overwritten:
- `ai-action/setup-ollama@v2` (line 28)
- `ai-action/setup-ollama@v2` (line 32)
- `actions/cache@v6` (line 37)

Locations:

- `action.yml:28`
- `action.yml:32`
- `action.yml:37`

### script-injection (severity: high)

Rule (b) violation: In the 'Run model' step, the env vars $MODEL (sourced from `inputs.model`) and $PROMPT (sourced from `inputs.prompt`) are used unquoted in the shell command `ollama run $MODEL "$(printf '%q' $PROMPT)"`. An unquoted `$MODEL` allows shell word-splitting and glob expansion of attacker-controlled input. Similarly, `$PROMPT` is unquoted inside the command substitution `$(printf '%q' $PROMPT)`, allowing shell metacharacters (`;`, `|`, `&`, backticks, `$(...)`) in the input to be interpreted by the shell before `printf` ever sees them. Both variables must be double-quoted: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"` .

Locations:

- `action.yml:48`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned ai-action/setup-ollama@v2 (both occurrences) to SHA bb1a15e4315698e668874dc3aad323d7c6d57ede # v2. 2. Pinned actions/cache@v6 to SHA 55cc8345863c7cc4c66a329aec7e433d2d1c52a9 # v6. 3. Fixed script injection in the 'Run model' step by double-quoting $MODEL and $PROMPT: changed `ollama run $MODEL "$(printf '%q' $PROMPT)"` to `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`.

