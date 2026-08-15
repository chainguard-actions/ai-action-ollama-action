<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the tag is moved.

In action.yml:
- `uses: ai-action/setup-ollama@v2` (line 27)
- `uses: ai-action/setup-ollama@v2` (line 31)
- `uses: actions/cache@v6` (line 35)

In .github/workflows/commitlint.yml:
- `uses: remarkablemark/commitlint@v2` (line 9)

In .github/workflows/release-please.yml:
- `uses: googleapis/release-please-action@v5` (line 22)
- `uses: actions/checkout@v7` (line 33)

In .github/workflows/test.yml:
- `uses: actions/checkout@v7` (line 14)

All should be pinned to a full SHA, e.g. `uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:35`
- `.github/workflows/commitlint.yml:9`
- `.github/workflows/release-please.yml:22`
- `.github/workflows/release-please.yml:33`
- `.github/workflows/test.yml:14`

### script-injection (severity: high)

Sub-rule (a): `${{ ... }}` expressions from `needs.*.outputs.*` are interpolated directly inside `run:` shell command strings in release-please.yml. These values go through YAML template substitution before the shell sees them, allowing shell metacharacter injection if the upstream release-please action produces unexpected output.

Offending lines in the 'Tag major and minor versions' step:
  `git tag -d v${{ needs.release.outputs.major }} || true`
  `git tag -d v${{ needs.release.outputs.major }}.${{ needs.release.outputs.minor }} || true`
  `git tag -a v${{ needs.release.outputs.major }} -m 'Release v${{ needs.release.outputs.major }}'`
  `git push -f origin v${{ needs.release.outputs.major }}`
  etc.

Offending line in the 'Tag latest release' step:
  `run: gh release edit ${{ needs.release.outputs.tag_name }} --latest`

Fix: move the values into `env:` variables and reference them as quoted shell variables, e.g. `"$TAG_NAME"`.

Locations:

- `.github/workflows/release-please.yml:43`
- `.github/workflows/release-please.yml:52`

### script-injection (severity: high)

Sub-rule (b): In action.yml's 'Run model' step, the shell variable `$MODEL` (sourced from `inputs.model`, an untrusted caller-controlled input) is expanded **unquoted** in the `run:` script:

  `ollama run $MODEL "$(printf '%q' $PROMPT)"`

An unquoted expansion allows the shell to parse metacharacters (`;`, `|`, `&`, whitespace, glob chars) out of the value, enabling command injection. `$MODEL` must be double-quoted: `ollama run "$MODEL" ...`.

Locations:

- `action.yml:46`

### github-env-injection (severity: high)

In action.yml's 'Run model' step, the output of `ollama run $MODEL "$(printf '%q' $PROMPT)"` is written to `$GITHUB_OUTPUT` via a heredoc without first sanitizing the value with `printf '%s' ... | tr -d '\n\r'`. The `MODEL` and `PROMPT` env vars are set from `inputs.model` and `inputs.prompt` respectively — both are caller-controlled untrusted inputs. A newline embedded in the model's response (or injected via the inputs) could break out of the heredoc delimiter and inject arbitrary key=value pairs into `$GITHUB_OUTPUT`.

The heredoc EOF delimiter (`EOF_OLLAMA_<random>`) mitigates some risk but does not replace the required sanitization of the inputs themselves before they influence the write.

Locations:

- `action.yml:41`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 4 findings across 4 files:

1. **unpinned-uses** (action.yml, commitlint.yml, release-please.yml, test.yml): Pinned all 7 mutable tag references to full 40-character commit SHAs with tag comments preserved:
   - ai-action/setup-ollama@v2 → @591531fff1153f90ea7a512c1c528231856f1036 # v2 (2 occurrences in action.yml)
   - actions/cache@v6 → @55cc8345863c7cc4c66a329aec7e433d2d1c52a9 # v6
   - remarkablemark/commitlint@v2 → @c4f9e77786e0c94d17570ed36be2adcf022711d3 # v2
   - googleapis/release-please-action@v5 → @45996ed1f6d02564a971a2fa1b5860e934307cf7 # v5
   - actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1 # v7 (2 occurrences)

2. **script-injection (a)** (release-please.yml): Moved all `${{ needs.release.outputs.* }}` expressions from inline run: shell strings into step-level env: blocks (MAJOR, MINOR, TAG_NAME) and referenced them as double-quoted shell variables.

3. **script-injection (b)** (action.yml): Double-quoted `$MODEL` → `"$SAFE_MODEL"` and `$PROMPT` → `"$SAFE_PROMPT"` in the ollama run command.

4. **github-env-injection** (action.yml): Added sanitization of MODEL and PROMPT inputs using `printf '%s' "$VAR" | tr -d '\n\r'` before they are used in the ollama command and heredoc write to $GITHUB_OUTPUT, preventing newline injection attacks.

