<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved or overwritten:
- `ai-action/setup-ollama@v2` (Setup Ollama step)
- `ai-action/setup-ollama@v2` (Setup Ollama v${{ inputs.version }} step)
- `actions/cache@v6` (Cache model step)

Locations:

- `action.yml:24`
- `action.yml:29`
- `action.yml:35`

### script-injection (severity: high)

Sub-rule (b): In the 'Run model' step, two env vars sourced from `inputs.*` are expanded unquoted inside the `run:` shell script:
1. `ollama run $MODEL` — `$MODEL` (from `inputs.model`) is unquoted, allowing shell metacharacter injection (`;`, `|`, `&`, etc.).
2. `$(printf '%q' $PROMPT)` — `$PROMPT` (from `inputs.prompt`) is unquoted inside a command substitution, allowing word-splitting and glob expansion before `printf` even sees the value.
Both should be double-quoted: `"$MODEL"` and `"$PROMPT"`.

Locations:

- `action.yml:50`

### github-env-injection (severity: high)

The 'Run model' step writes to `$GITHUB_OUTPUT` using a heredoc block that includes content derived from `inputs.model` (`$MODEL`) and `inputs.prompt` (`$PROMPT`) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`) applied before the write. Although the EOF delimiter is randomized (protecting against delimiter injection), the `response` value written to `$GITHUB_OUTPUT` is not sanitized, and the inputs themselves are not stripped of newline/carriage-return characters before use. An attacker-controlled `inputs.model` or `inputs.prompt` containing embedded newlines could inject additional key=value pairs into `$GITHUB_OUTPUT`.

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:
1. unpinned-uses: Pinned ai-action/setup-ollama@v2 to @bb1a15e4315698e668874dc3aad323d7c6d57ede # v2 (both occurrences) and actions/cache@v6 to @55cc8345863c7cc4c66a329aec7e433d2d1c52a9 # v6.
2. script-injection: Added SAFE_MODEL and SAFE_PROMPT variables; quoted both as "$SAFE_MODEL" and "$SAFE_PROMPT" in the ollama run command and printf call.
3. github-env-injection: Sanitized MODEL and PROMPT inputs by stripping newline/carriage-return characters (printf '%s' "$VAR" | tr -d '\n\r') before use, preventing attacker-controlled newlines from injecting additional key=value pairs into $GITHUB_OUTPUT. The heredoc format is preserved for multi-line ollama response output.

