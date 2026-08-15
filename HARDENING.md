<!-- markdownlint-disable -->

# Hardening Report: ai-action--ollama-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ai-action--ollama-action/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags instead of full 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if the tag is moved. Failing references: action.yml — `ai-action/setup-ollama@v2` (×2), `actions/cache@v5`; commitlint.yml — `remarkablemark/commitlint@v2`; release-please.yml — `googleapis/release-please-action@v4`, `actions/checkout@v6`; test.yml — `actions/checkout@v6`.

Locations:

- `action.yml:24`
- `action.yml:29`
- `action.yml:34`
- `.github/workflows/commitlint.yml:10`
- `.github/workflows/release-please.yml:22`
- `.github/workflows/release-please.yml:33`
- `.github/workflows/test.yml:16`

### script-injection (severity: high)

Rule (a): In release-please.yml, `${{ needs.release.outputs.major }}`, `${{ needs.release.outputs.minor }}`, and `${{ needs.release.outputs.tag_name }}` are directly interpolated inside `run:` shell command strings. These are `steps.*.outputs.*` values that flow through YAML template substitution before the shell processes them, allowing injection of shell metacharacters. Offending lines include: `git tag -d v${{ needs.release.outputs.major }} || true`, `git tag -a v${{ needs.release.outputs.major }} -m 'Release v${{ needs.release.outputs.major }}'`, and `gh release edit ${{ needs.release.outputs.tag_name }} --latest`. Rule (b): In action.yml, the `run:` block expands `$MODEL` and `$PROMPT` unquoted — `ollama run $MODEL "$(printf '%q' $PROMPT)"` — where both env vars are sourced from `inputs.model` and `inputs.prompt`. An unquoted `$MODEL` allows shell word-splitting and glob expansion on attacker-controlled input.

Locations:

- `.github/workflows/release-please.yml:38`
- `.github/workflows/release-please.yml:39`
- `.github/workflows/release-please.yml:40`
- `.github/workflows/release-please.yml:41`
- `.github/workflows/release-please.yml:42`
- `.github/workflows/release-please.yml:43`
- `.github/workflows/release-please.yml:46`
- `action.yml:50`

### github-env-injection (severity: high)

In action.yml, the `run:` block writes to `$GITHUB_OUTPUT` using values derived from `inputs.model` (via `$MODEL`) and `inputs.prompt` (via `$PROMPT`) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`) before the write. The heredoc delimiter is randomized (good), but the content written — including the `response<<$EOF` line and the ollama output — is constructed from unsanitized user-controlled inputs. A newline embedded in `$MODEL` or `$PROMPT` could inject additional key=value pairs into `$GITHUB_OUTPUT`, poisoning downstream step outputs.

Locations:

- `action.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings:

1. unpinned-uses: Pinned all 7 mutable tag references to full 40-char SHAs with tag comments preserved:
   - action.yml: ai-action/setup-ollama@v2 (×2) → @591531fff1153f90ea7a512c1c528231856f1036 # v2
   - action.yml: actions/cache@v5 → @caa296126883cff596d87d8935842f9db880ef25 # v5
   - commitlint.yml: remarkablemark/commitlint@v2 → @c4f9e77786e0c94d17570ed36be2adcf022711d3 # v2
   - release-please.yml: googleapis/release-please-action@v4 → @5c625bfb5d1ff62eadeeb3772007f7f66fdcf071 # v4
   - release-please.yml + test.yml: actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803 # v6

2. script-injection: In release-please.yml, moved all ${{ needs.release.outputs.* }} expressions into env: blocks (MAJOR, MINOR, TAG_NAME) and referenced them as double-quoted shell variables. In action.yml, fixed unquoted $MODEL to use sanitized "$safe_model".

3. github-env-injection: In action.yml, added sanitization of $MODEL and $PROMPT via printf '%s' | tr -d '\n\r' before use, preventing newline injection into $GITHUB_OUTPUT.

