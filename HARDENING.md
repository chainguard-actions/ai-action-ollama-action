<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **ai-action--ollama-action/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three 'uses:' references in action.yml use mutable tags instead of pinned 40-character SHA commit digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved:
- 'ai-action/setup-ollama@v2' (Setup Ollama step, line 27)
- 'ai-action/setup-ollama@v2' (Setup Ollama v${{ inputs.version }} step, line 33)
- 'actions/cache@v5' (Cache model step, line 39)

Locations:

- `action.yml:27`
- `action.yml:33`
- `action.yml:39`

### script-injection (severity: high)

Rule (b) violation: In the 'Run model' step, the shell variable $MODEL (sourced from inputs.model via the env: block) is used unquoted in the run: command: `ollama run $MODEL "$(printf '%q' $PROMPT)"`. An unquoted shell variable allows an attacker-controlled input.model value containing shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) to be interpreted by the shell, enabling command injection. $MODEL must be double-quoted: `ollama run "$MODEL" ...`.

Locations:

- `action.yml:48`

### github-env-injection (severity: high)

In the 'Run model' step, the value of $PROMPT (sourced from inputs.prompt via the env: block) is written to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' "$PROMPT" | tr -d '\n\r'`). The heredoc delimiter is randomized (mitigating direct heredoc injection), but the raw unsanitized prompt value is still appended to $GITHUB_OUTPUT. A prompt containing newlines could inject additional key=value pairs into the output file. The value must be sanitized before writing.

Locations:

- `action.yml:46`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in action.yml:
1. unpinned-uses: Pinned 'ai-action/setup-ollama@v2' to SHA d0eb66d576e0880178be72b68b77b2dbbf5a884f (both occurrences at lines 27 and 33) and 'actions/cache@v5' to SHA 27d5ce7f107fe9357f9df03efb73ab90386fccae (line 39). Original tags preserved as comments.
2. script-injection: Double-quoted $MODEL in the 'ollama run' command (was unquoted, allowing shell metacharacter injection via attacker-controlled model name).
3. github-env-injection: Added sanitization of $PROMPT using 'printf "%s" "$PROMPT" | tr -d '\n\r'' before writing to $GITHUB_OUTPUT, preventing newline injection of additional key=value pairs into the output file.

