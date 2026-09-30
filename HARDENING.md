<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tags are moved:
- `uses: ai-action/setup-ollama@v2` (Setup Ollama step)
- `uses: ai-action/setup-ollama@v2` (Setup Ollama v${{ inputs.version }} step)
- `uses: actions/cache@v6` (Cache model step)
All should be pinned to full commit SHAs, e.g. `uses: actions/cache@<40-hex-sha> # v6`.

Locations:

- `action.yml:24`
- `action.yml:29`
- `action.yml:34`

### script-injection (severity: high)

Rule (b) violation in the 'Run model' step: the env vars `$MODEL` and `$PROMPT` — sourced from `inputs.model` and `inputs.prompt` respectively — are expanded unquoted inside the `run:` shell script. Specifically:
- `ollama run $MODEL` — `$MODEL` is completely unquoted, allowing shell metacharacter injection (`;`, `|`, `&`, etc.).
- `$(printf '%q' $PROMPT)` — `$PROMPT` is unquoted inside the command substitution.
These should be double-quoted: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"` (or the guarded form for optional inputs).

Locations:

- `action.yml:47`

### github-env-injection (severity: high)

Rule (d) violation in the 'Run model' step: the env vars `MODEL` and `PROMPT` are set from `inputs.model` and `inputs.prompt` (untrusted user-controlled inputs) and their values flow into the heredoc block written to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled newline in `inputs.model` or `inputs.prompt` could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting other outputs. The `EOF` delimiter itself is randomized (good), but the content written — the output of `ollama run $MODEL ...` — is still derived from unsanitized inputs.

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:

1. unpinned-uses: Pinned ai-action/setup-ollama@v2 → @5f3116a5cd85594f351c3f8600a5ac08e63bc285 # v2 (both occurrences) and actions/cache@v6 → @55cc8345863c7cc4c66a329aec7e433d2d1c52a9 # v6.

2. script-injection: Added double-quoting around $MODEL and $PROMPT in the 'Run model' step shell script. Changed `ollama run $MODEL "$(printf '%q' $PROMPT)"` to `ollama run "$safe_model" "$(printf '%q' "$safe_prompt")"` using sanitized variables.

3. github-env-injection: Added sanitization of the MODEL and PROMPT env vars (sourced from untrusted inputs) using `printf '%s' "$VAR" | tr -d '\n\r'` before they are used in the command that produces output written to $GITHUB_OUTPUT. This prevents newline injection attacks. The randomized EOF heredoc delimiter is preserved for safe multi-line response capture.

