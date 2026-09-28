<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v1.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action step `uses: ai-action/setup-ollama@v1` references a mutable tag (`v1`) rather than a pinned 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling a supply-chain attack.

Locations:

- `action.yml:24`

### script-injection (severity: high)

Rule (b) violation: In the 'Run LLM' step, the env vars `$MODEL` and `$PROMPT` (both sourced from `inputs.*` via the `env:` block) are expanded unquoted inside the `run:` shell script. Specifically: `ollama run $MODEL "$(printf '%q' $PROMPT)"` — `$MODEL` is completely unquoted, and `$PROMPT` is unquoted inside the command substitution `$(...)`. An attacker-controlled input containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) can break out of the intended command and execute arbitrary shell commands.

Locations:

- `action.yml:35`

### github-env-injection (severity: high)

The 'Run LLM' step writes to `$GITHUB_OUTPUT` (`} >> $GITHUB_OUTPUT`) without sanitizing the inputs first. The env vars `MODEL` and `PROMPT` are set from `inputs.model` and `inputs.prompt` respectively — both are untrusted inputs. `$MODEL` is passed unquoted to `ollama run`, and neither value is passed through `printf '%s' ... | tr -d '\n\r'` before the write. A value containing newline characters in `$MODEL` or in the ollama response derived from `$PROMPT` can corrupt the `$GITHUB_OUTPUT` file format, potentially injecting arbitrary key=value pairs into the output environment.

Locations:

- `action.yml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

1. Pinned ai-action/setup-ollama@v1 to commit SHA bfc77222729bdd88f91b95ef1142d247773e486d (kept # v1 comment for readability). 2. Fixed script-injection: $MODEL is now sanitized and double-quoted as "$safe_model", and $PROMPT is now double-quoted as "$PROMPT" (was unquoted inside a command substitution with printf '%q'). 3. Fixed github-env-injection: sanitized $MODEL with `printf '%s' "$MODEL" | tr -d '\n\r'` before use to strip newlines that could corrupt GITHUB_OUTPUT format; also properly quoted $GITHUB_OUTPUT variable reference.

