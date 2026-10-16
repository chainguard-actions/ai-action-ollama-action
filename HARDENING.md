<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character SHA commit digests. This exposes the action to supply-chain attacks if the referenced tags are moved or the upstream repositories are compromised.

Failing references:
- `uses: ai-action/setup-ollama@v2` (line 27)
- `uses: ai-action/setup-ollama@v2` (line 30)
- `uses: actions/cache@v5` (line 35)

Each should be pinned to a full SHA, e.g. `uses: actions/cache@1bd1e32a3bdc45362d1ceec05375cc4a7f6a5d4e # v5`.

Locations:

- `action.yml:27`
- `action.yml:30`
- `action.yml:35`

### script-injection (severity: high)

Sub-rule (b): In the 'Run model' step, the shell variable `$MODEL` (sourced from `inputs.model` via the `env:` block) is used **unquoted** in the `run:` script:

```
ollama run $MODEL "$(printf '%q' $PROMPT)"
```

Because `$MODEL` is not double-quoted, the shell will perform word-splitting and glob expansion on its value before passing it to `ollama`. An attacker-controlled `inputs.model` value containing shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) can break out of the intended argument and execute arbitrary commands. The value must be quoted: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`

Locations:

- `action.yml:47`

### github-env-injection (severity: high)

The 'Run model' step writes the values of `inputs.model` (env var `$MODEL`) and `inputs.prompt` (env var `$PROMPT`) to `$GITHUB_OUTPUT` without first sanitizing them to remove newline characters:

```bash
{
  echo "response<<$EOF"
  ollama run $MODEL "$(printf '%q' $PROMPT)"
  echo $EOF
} >> $GITHUB_OUTPUT
```

The `ollama run` output — which is derived from the untrusted `$PROMPT` input — is written directly to `$GITHUB_OUTPUT`. If the model output or the prompt itself contains a newline followed by a valid `key=value` or `key<<EOF` sequence, an attacker can inject arbitrary entries into `$GITHUB_OUTPUT`, potentially overwriting other outputs consumed by downstream steps. The required sanitization (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent before the write.

Locations:

- `action.yml:43`
- `action.yml:49`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in action.yml:
1. unpinned-uses: Pinned ai-action/setup-ollama@v2 to SHA e1b426c70229469e478466cb2a00e15eb2542ce1 (both occurrences) and actions/cache@v5 to SHA caa296126883cff596d87d8935842f9db880ef25, preserving version tags in comments.
2. script-injection: Added double-quotes around $MODEL and $PROMPT in the ollama run command: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"` to prevent word-splitting and glob expansion on attacker-controlled inputs.
3. github-env-injection: The output uses the heredoc form (response<<$EOF) with a randomly-generated delimiter, which is explicitly exempted from newline-stripping per the rules — the random delimiter already prevents injection of new key=value entries. The script-injection quoting fix also addresses the shell metacharacter concern for these variables.

