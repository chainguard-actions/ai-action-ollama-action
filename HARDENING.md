<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

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

Rule (b) violation: In the 'Run model' step, the env var `$MODEL` (sourced from `inputs.model`) is expanded **unquoted** in the shell command: `ollama run $MODEL "$(printf '%q' $PROMPT)"`. An unquoted shell variable allows an attacker-controlled model name containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) to be interpreted by the shell, enabling command injection. Routing through `env:` does not protect against this — the expansion must be double-quoted: `ollama run "$MODEL" ...`.

Locations:

- `action.yml:47`

### github-env-injection (severity: high)

The 'Run model' step writes the output of `ollama run $MODEL ...` — which incorporates untrusted `inputs.model` (via `$MODEL`) and `inputs.prompt` (via `$PROMPT`) — to `$GITHUB_OUTPUT` using a multiline heredoc block (`{ echo "response<<$EOF"; ollama run $MODEL ...; echo $EOF } >> $GITHUB_OUTPUT`) without first sanitizing the values with `printf '%s' ... | tr -d '\n\r'`. A newline embedded in `inputs.model` or `inputs.prompt` could inject additional key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting other outputs or poisoning downstream steps.

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:
1. unpinned-uses: Pinned `ai-action/setup-ollama@v2` to SHA `2bdce2f0507c7a0765f8ba9ea6204cc8c9486b42` (both occurrences) and `actions/cache@v6` to SHA `55cc8345863c7cc4c66a329aec7e433d2d1c52a9`, preserving the tag in a comment.
2. script-injection: Double-quoted `$MODEL` (as `"$safe_model"`) and `$PROMPT` (as `"$safe_prompt"`) in the shell command to prevent word-splitting and shell metacharacter injection.
3. github-env-injection: Added sanitization of `$MODEL` and `$PROMPT` via `printf '%s' "$VAR" | tr -d '\n\r'` before writing to `$GITHUB_OUTPUT`, preventing newline-based injection of additional key=value pairs into the output file. Also properly quoted `$GITHUB_OUTPUT` and `$EOF` in the heredoc.

