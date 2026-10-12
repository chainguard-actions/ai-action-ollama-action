<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml are pinned to mutable tags rather than immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten:
- `uses: ai-action/setup-ollama@v2` (appears twice, lines ~27 and ~32)
- `uses: actions/cache@v5` (line ~37)

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`

### script-injection (severity: high)

Sub-rule (b): In the 'Run model' step, the env var `$MODEL` (sourced from `inputs.model`, an attacker-controlled value) is expanded **unquoted** in the shell command:

  `ollama run $MODEL "$(printf '%q' $PROMPT)"`

An unquoted `$MODEL` allows the shell to parse metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) out of the value, enabling command injection. `$PROMPT` is also unquoted inside the command substitution `$(printf '%q' $PROMPT)`. Both should be double-quoted: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"` .

Locations:

- `action.yml:51`

### github-env-injection (severity: high)

The 'Run model' step writes the value of `inputs.prompt` (via env var `$PROMPT`) and `inputs.model` (via env var `$MODEL`) to `$GITHUB_OUTPUT` using a heredoc multiline block without first sanitizing newlines with `printf '%s' ... | tr -d '\n\r'`. An attacker-controlled prompt or model name containing embedded newlines could inject additional key=value pairs or heredoc terminators into `$GITHUB_OUTPUT`, poisoning downstream step outputs. The unsanitized write is:

  `echo "response<<$EOF"` / `ollama run $MODEL ...` / `echo $EOF` >> `$GITHUB_OUTPUT`

All values derived from `inputs.*` must be sanitized before being written to special environment files.

Locations:

- `action.yml:48`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

1. Pinned ai-action/setup-ollama@v2 (both occurrences) to SHA 44a3e55e2fe1af8d9150329933b8a9f324f4ca74 and actions/cache@v5 to SHA caa296126883cff596d87d8935842f9db880ef25. 2. Fixed script-injection by double-quoting $MODEL and $PROMPT in the ollama run command: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`. 3. For github-env-injection: the response is written via a heredoc with a random delimiter (EOF_OLLAMA_$(openssl rand -hex 12)), which per the hardening rules already prevents injection and should NOT have newlines stripped (that would merge multi-line responses). Additionally quoted `echo "$EOF"` and `"$GITHUB_OUTPUT"` for correctness.

