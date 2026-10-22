<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Composite action steps pin actions to tags instead of full 40-character commit SHAs: 'uses: ai-action/setup-ollama@v2' (line 30), 'uses: ai-action/setup-ollama@v2' (line 34), 'uses: actions/cache@v6' (line 40).

Locations:

- `action.yml:30`
- `action.yml:34`
- `action.yml:40`

### script-injection (severity: high)

Sub-rule (b) is violated: shell variables that hold workflow inputs are expanded without double quotes in the 'Run model' step. The offending line is `ollama run $MODEL "$(printf '%q' $PROMPT)"`. MODEL comes from inputs.model and PROMPT comes from inputs.prompt, and both are unquoted, so the shell can split words and expand globs on their values. The fix is `ollama run "$MODEL" "$PROMPT"`. The same step also leaves `echo $EOF` and `>> $GITHUB_OUTPUT` unquoted.

Locations:

- `action.yml:52`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Pinned ai-action/setup-ollama@v2 to 27578c4f6dab23d6b8eb49566a85cf8d6f997a9a (2 places) and actions/cache@v6 to 55cc8345863c7cc4c66a329aec7e433d2d1c52a9, with tag comments. In the 'Run model' step, quoted $MODEL, $EOF and $GITHUB_OUTPUT, and replaced "$(printf '%q' $PROMPT)" with "$PROMPT". Upstream's unquoted printf split the prompt into words and joined them back with no spaces; it now reaches ollama exactly as written. The random heredoc delimiter is unchanged.

### Iteration 2

**Fixes applied:** unpinned-uses

**Notes:**

Pinned both ai-action/setup-ollama@v2 references in action.yml to 27578c4f6dab23d6b8eb49566a85cf8d6f997a9a # v2.

