<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action references other actions by version tag instead of a full 40-character commit SHA: `uses: ai-action/setup-ollama@v2` (line 30), `uses: ai-action/setup-ollama@v2` (line 34), `uses: actions/cache@v6` (line 40).

Locations:

- `action.yml:30`
- `action.yml:34`
- `action.yml:40`

### script-injection (severity: high)

Sub-rule (b), unquoted shell expansion of untrusted data. In the 'Run model' step, MODEL and PROMPT come from inputs.model and inputs.prompt, but the script expands them without double quotes: `ollama run $MODEL "$(printf '%q' $PROMPT)"`. Bare $MODEL is split on whitespace and glob-expanded, so it can inject extra arguments. Bare $PROMPT inside the command substitution is also split and globbed. The fix is to quote both: `ollama run "$MODEL" "$(printf '%q' "$PROMPT")"` (or pass "$PROMPT" directly).

Locations:

- `action.yml:52`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Pinned both ai-action/setup-ollama@v2 references to cceadd0155f199c7b19f04ab5a1cae9e0c1af005 and actions/cache@v6 to 55cc8345863c7cc4c66a329aec7e433d2d1c52a9, keeping the tag as a trailing comment. In the 'Run model' step I quoted the variables as the finding suggested: ollama run "$MODEL" "$(printf '%q' "$PROMPT")". I also quoted "$EOF" and "$GITHUB_OUTPUT". The heredoc delimiter is already random (openssl rand), so I left it as it was. One behaviour change: before, printf '%q' got each word of the prompt as a separate argument and joined them with no space between them. Now it gets the whole prompt as one argument, so spaces in the prompt are kept, escaped with backslashes. Nothing was run to test this.

### Iteration 2

**Fixes applied:** unpinned-uses

**Notes:**

Put back `version: ${{ inputs.version }}` for the setup-ollama step, which had been corrupted. Pinned actions/cache to 55cc8345863c7cc4c66a329aec7e433d2d1c52a9 # v6 (looked up with lookup_action_sha).

