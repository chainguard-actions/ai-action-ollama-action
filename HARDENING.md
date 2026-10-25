<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action steps use tag refs instead of full 40-character commit SHAs: 'uses: ai-action/setup-ollama@v2' (twice) and 'uses: actions/cache@v6'.

Locations:

- `action.yml:30`
- `action.yml:34`
- `action.yml:40`

### script-injection (severity: high)

Rule (b), unquoted expansion of untrusted data. In the 'Run model' step, MODEL (from inputs.model) and PROMPT (from inputs.prompt) are expanded without double quotes: `ollama run $MODEL "$(printf '%q' $PROMPT)"`. The shell splits these values on whitespace and expands glob characters in them, so a caller can inject extra arguments into ollama. The fix is `ollama run "$MODEL" "$PROMPT"`. Also, `$GITHUB_OUTPUT` and `$EOF` are unquoted.

Locations:

- `action.yml:52`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Pinned ai-action/setup-ollama@v2 (both uses) to d01d384a4092ed43e2cfea696ad74c24b53b2ca5 and actions/cache@v6 to 55cc8345863c7cc4c66a329aec7e433d2d1c52a9, with the tags kept as trailing comments. In the 'Run model' step, the line is now `ollama run "$MODEL" "$PROMPT"` and $EOF and $GITHUB_OUTPUT are quoted. This changes the output: upstream's `printf '%q' $PROMPT` split the prompt into words and joined them with no spaces, and escaped shell characters with backslashes. The prompt now reaches ollama exactly as the caller wrote it. Nothing was run to test this.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

In the 'Run model' step, replaced `ollama run $MODEL "$(printf '%q' $PROMPT)"` with `ollama run "$MODEL" "$PROMPT"` so the model and prompt each reach ollama as one argument with no word-splitting or globbing. Also quoted $EOF and $GITHUB_OUTPUT. The heredoc delimiter is still random per run (openssl rand).

