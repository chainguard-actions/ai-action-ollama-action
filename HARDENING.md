<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references actions by mutable tags instead of full 40-character commit SHAs: line 30 `uses: ai-action/setup-ollama@v2`, line 34 `uses: ai-action/setup-ollama@v2`, line 40 `uses: actions/cache@v6`.

Locations:

- `action.yml:30`
- `action.yml:34`
- `action.yml:40`

### script-injection (severity: high)

Sub-rule (b), unquoted expansion of untrusted env vars. The 'Run model' step routes inputs.model and inputs.prompt through env as MODEL and PROMPT, then expands them unquoted on line 52: `ollama run $MODEL "$(printf '%q' $PROMPT)"`. Both $MODEL and the $PROMPT inside the command substitution are unquoted, so the shell splits them on whitespace and expands glob characters. They should be written as "$MODEL" and "$PROMPT".

Locations:

- `action.yml:52`

### github-env-injection (severity: high)

The 'Run model' step writes output derived from untrusted input to $GITHUB_OUTPUT with no `printf '%s' ... | tr -d '\n\r'` sanitization (lines 50-54: `{ echo "response<<$EOF"; ollama run $MODEL "$(printf '%q' $PROMPT)"; echo $EOF; } >> $GITHUB_OUTPUT`). The model response comes from inputs.prompt and inputs.model. A random heredoc delimiter makes injection harder but does not meet the required sanitization. $GITHUB_OUTPUT and $EOF are also unquoted.

Locations:

- `action.yml:54`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Pinned ai-action/setup-ollama@v2 (0ee037bc...) and actions/cache@v6 (55cc8345...) to SHAs with tag comments. Quoted $MODEL and $PROMPT and pass the prompt directly as a single argument (the old unquoted printf '%q' $PROMPT split it into words and glued them back together without spaces). The response is captured first, so under bash -e a failing ollama run still stops the step. It is then written inside a heredoc block whose delimiter comes from openssl rand, so a value cannot guess it. The delimiter, $GITHUB_OUTPUT and the response are quoted. Newlines are not stripped because the response is multi-line and the random delimiter is the correct safeguard for that form.

