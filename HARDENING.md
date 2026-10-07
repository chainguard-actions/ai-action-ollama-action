<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml are pinned to mutable tags instead of immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten:
- `ai-action/setup-ollama@v2` (line 27)
- `ai-action/setup-ollama@v2` (line 31)
- `actions/cache@v6` (line 35)
These should be pinned to full SHA digests, e.g. `actions/cache@<40-hex-sha> # v6`.

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:35`

### script-injection (severity: high)

Rule (b) violation in the 'Run model' step: the shell variables `$MODEL` and `$PROMPT` — both sourced from attacker-controlled `inputs.*` values via the `env:` block — are expanded **unquoted** inside the `run:` script.

1. `ollama run $MODEL ...` (line 48): `$MODEL` is unquoted. A model name containing shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) will be word-split and interpreted by the shell, enabling command injection.
2. `"$(printf '%q' $PROMPT)"` (line 48): `$PROMPT` is unquoted inside the command substitution. The shell word-splits `$PROMPT` before passing it to `printf`, so a prompt containing spaces or glob characters is split into multiple arguments, breaking the quoting intent.

Fix: quote all expansions — `"$MODEL"` and `"$PROMPT"` — so the shell treats them as single tokens:
```bash
ollama run "$MODEL" "$(printf '%q' "$PROMPT")"
```

Locations:

- `action.yml:48`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed three unpinned `uses:` references by resolving their full SHA digests: `ai-action/setup-ollama@v2` → `@2bdce2f0507c7a0765f8ba9ea6204cc8c9486b42 # v2` (both occurrences), and `actions/cache@v6` → `@55cc8345863c7cc4c66a329aec7e433d2d1c52a9 # v6`. Fixed script injection by quoting both `$MODEL` and `$PROMPT` in the `run:` script: changed `ollama run $MODEL "$(printf '%q' $PROMPT)"` to `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"`.

