<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tags are moved or overwritten:
- `uses: ai-action/setup-ollama@v2` (Setup Ollama step, line 27)
- `uses: ai-action/setup-ollama@v2` (Setup Ollama v${{ inputs.version }} step, line 32)
- `uses: actions/cache@v5` (Cache model step, line 38)

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:38`

### script-injection (severity: high)

Sub-rule (b) violation in the 'Run model' step: env vars sourced from `inputs.*` are expanded unquoted inside the `run:` shell script.

1. `$MODEL` (from `inputs.model`) is unquoted: `ollama run $MODEL ...` — an attacker-controlled model name containing shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, globs) will be parsed by the shell before execution.

2. `$PROMPT` (from `inputs.prompt`) is unquoted inside a command substitution: `$(printf '%q' $PROMPT)` — word-splitting and glob expansion occur on `$PROMPT` before `printf` ever sees it, allowing an attacker to inject additional arguments or break the quoting.

Both variables must be double-quoted: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"` .

Locations:

- `action.yml:49`

### github-env-injection (severity: high)

The 'Run model' step writes to `$GITHUB_OUTPUT` without sanitizing the value first. The block written is:

```
echo "response<<$EOF"
ollama run $MODEL "$(printf '%q' $PROMPT)"
echo $EOF
```

The response value is the output of `ollama run`, which is driven by `inputs.model` and `inputs.prompt` — both untrusted inputs. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write. While the randomized `$EOF` delimiter (`EOF_OLLAMA_$(openssl rand -hex 12)`) prevents delimiter injection, the response content itself is written unsanitized to `$GITHUB_OUTPUT`, allowing a crafted model response to inject additional key=value pairs into the output file and potentially poison downstream steps.

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:
1. unpinned-uses: Pinned ai-action/setup-ollama@v2 to SHA 6782d243db5fbcbf7583532ce0b5122f64242689 (both occurrences at lines 27 and 32) and actions/cache@v5 to SHA caa296126883cff596d87d8935842f9db880ef25 (line 38), preserving original tags in comments.
2. script-injection: Double-quoted $MODEL and $PROMPT in the 'Run model' step shell script to prevent word-splitting and glob expansion on attacker-controlled values.
3. github-env-injection: Captured ollama output into raw_response variable, then sanitized with `tr -d '\n\r'` into safe_response before writing to $GITHUB_OUTPUT, preventing newline injection into the output file.

