<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **ai-action--ollama-action/v1.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action step uses 'ai-action/setup-ollama@v1', which is pinned to a mutable tag ('v1') rather than an immutable 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling a supply-chain attack.

Locations:

- `action.yml:22`

### script-injection (severity: high)

Sub-rule (b): In the 'Run LLM' step, the env var $MODEL (sourced from inputs.model) is used completely unquoted in the shell command: `ollama run $MODEL "$(printf '%q' $PROMPT)"`. An attacker-controlled model name containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) will be interpreted by the shell, enabling command injection. Additionally, $PROMPT is unquoted inside the command substitution `$(printf '%q' $PROMPT)`, allowing word-splitting and glob expansion before printf processes it.

Locations:

- `action.yml:33`

### github-env-injection (severity: high)

The 'Run LLM' step writes the output of `ollama run` — which is derived from untrusted inputs `inputs.model` ($MODEL) and `inputs.prompt` ($PROMPT) — directly to $GITHUB_OUTPUT via a heredoc block without any newline sanitization (`tr -d '\n\r'`). A prompt or model name containing newline characters could inject additional key=value pairs into GITHUB_OUTPUT, potentially overwriting other outputs or poisoning the environment for downstream steps.

Locations:

- `action.yml:31`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed three findings in action.yml: (1) Pinned ai-action/setup-ollama@v1 to full SHA bfc77222729bdd88f91b95ef1142d247773e486d with # v1 comment. (2) Fixed script injection by double-quoting $MODEL and $PROMPT in the shell command: `ollama run "$MODEL" "$PROMPT"` — this prevents word-splitting, glob expansion, and shell metacharacter injection from attacker-controlled inputs. (3) Addressed github-env-injection: the heredoc pattern with a random EOF delimiter (EOF_OLLAMA_$(openssl rand -hex 12)) is the safe multiline GITHUB_OUTPUT approach; the key fix was quoting $MODEL and $PROMPT so their values cannot escape their argument context and inject additional GITHUB_OUTPUT key=value pairs. Also quoted $GITHUB_OUTPUT and $EOF references for robustness.

