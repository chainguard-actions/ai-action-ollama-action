<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved:
- `ai-action/setup-ollama@v2` (line 27)
- `ai-action/setup-ollama@v2` (line 33)
- `actions/cache@v6` (line 40)

Locations:

- `action.yml:27`
- `action.yml:33`
- `action.yml:40`

### script-injection (severity: high)

Rule (b) violation: In the 'Run model' step, the shell variable `$MODEL` (sourced from `inputs.model` via the `env:` block) is used **unquoted** in the `run:` script: `ollama run $MODEL "$(printf '%q' $PROMPT)"`. An unquoted expansion allows shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) in the input value to be interpreted by the shell, enabling command injection. `$MODEL` must be double-quoted: `ollama run "$MODEL" ...`.

Locations:

- `action.yml:55`

### github-env-injection (severity: high)

In the 'Run model' step, the output of `ollama run $MODEL "$(printf '%q' $PROMPT)"` is written directly to `$GITHUB_OUTPUT` without sanitization. The values `$MODEL` and `$PROMPT` are derived from `inputs.model` and `inputs.prompt` respectively. Although the heredoc EOF delimiter is randomized (protecting against delimiter injection), the *content* written to `$GITHUB_OUTPUT` is not sanitized with `printf '%s' ... | tr -d '\n\r'` before the write. A newline embedded in the model's response (or injected via `inputs.model`) could inject additional key=value pairs into `$GITHUB_OUTPUT`.

Locations:

- `action.yml:49`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:
1. unpinned-uses: Pinned `ai-action/setup-ollama@v2` to SHA `6c9453881a29f2604c0ef56e20344141cc556dfb` (both occurrences) and `actions/cache@v6` to SHA `55cc8345863c7cc4c66a329aec7e433d2d1c52a9`, with original tags preserved as comments.
2. script-injection: Added double-quotes around `$MODEL` (via `$safe_model`) and `$PROMPT` in the `ollama run` command to prevent shell metacharacter interpretation.
3. github-env-injection: Sanitized the `$MODEL` input by stripping newlines/carriage returns with `printf '%s' "$MODEL" | tr -d '\n\r'` before use, preventing newline injection into `$GITHUB_OUTPUT` via the model name input.

