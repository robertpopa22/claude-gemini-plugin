---
description: Run a Gemini adversarial code review against local git state (working tree or branch diff). Mirrors /codex:review semantics — review-only, returns Gemini verdict verbatim.
argument-hint: '[--wait|--background] [--base <ref>] [--scope working-tree|branch] [--model <name>]'
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash(git:*), Bash(gemini:*), AskUserQuestion
---

Run a Gemini adversarial review against local git state.

Raw slash-command arguments: `$ARGUMENTS`

## Core constraint

- This command is **review-only**.
- Do not fix issues, apply patches, or suggest you are about to make changes.
- Your only job is to run the review and return Gemini's output verbatim to the user.

## Execution mode rules

- If `$ARGUMENTS` includes `--wait`, run foreground.
- If `$ARGUMENTS` includes `--background`, run in a Claude background task.
- Otherwise, estimate review size before asking:
  - For working-tree: `git status --short --untracked-files=all` + `git diff --shortstat --cached` + `git diff --shortstat`.
  - For branch: `git diff --shortstat <base>...HEAD` (default base = `main` or `master`).
  - Recommend `--wait` only when review is tiny (1-2 files, no broader change).
  - Otherwise recommend `--background` (Gemini Pro reviews can take 30-90s).
  - When in doubt, run the review.
- Use `AskUserQuestion` once with `Wait for results (Recommended)` or `Run in background`.

## Scope & base resolution

- `--scope working-tree` (default): review uncommitted changes (staged + unstaged + untracked).
- `--scope branch`: review committed changes against `--base` (default `main`).
- If no base supplied for branch scope, detect via: `git symbolic-ref refs/remotes/origin/HEAD | sed 's@.*/@@'` or fall back to `main` / `master`.

## Model selection

- Default model: `gemini-3-pro-preview` (top reasoning).
- Fallback if 404: `gemini-2.5-pro`.
- Override via `--model <name>` in `$ARGUMENTS`.

## Foreground flow

1. Compute the diff:
   - Working-tree: `git --no-pager diff HEAD && git ls-files --others --exclude-standard | xargs -I{} sh -c 'echo "--- NEW FILE: {} ---" && cat "{}"'`
   - Branch: `git --no-pager diff <base>...HEAD`
2. Pipe diff content to Gemini with structured prompt:

```bash
DIFF=$(<diff command>); echo "$DIFF" | gemini -m gemini-3-pro-preview -p "$REVIEW_PROMPT"
```

3. Review prompt (`$REVIEW_PROMPT`):

```
You are performing an adversarial code review on the diff piped via stdin.

Output format:

## Summary
One-paragraph verdict: SHIP / SHIP WITH CHANGES / DO NOT SHIP, plus headline reason.

## Top Issues
For each issue (max 10, sorted by severity):
- **[SEVERITY]** `<file>:<line>` — <one-line description>
  - Why it matters: <one sentence>
  - Fix: <concrete change, not generic advice>

Severities: CRITIC (security/data loss/correctness), HIGH (bug/regression), MEDIUM (smell/perf), LOW (style).

## Edge Cases Worth Testing
Bullet list of test scenarios the diff should be tested against.

## Final Verdict
One line.

Be terse. Cite file:line when possible. No fluff. No general advice — only diff-specific.
```

4. Return Gemini stdout verbatim. No commentary, no paraphrase, no summary.
5. Do NOT fix issues mentioned. Review-only.

## Background flow

- Launch foreground command with `run_in_background: true`.
- Do not call `BashOutput` or wait this turn.
- Tell user: "Gemini review started in background. Output file: `<path>`. Check back when notified."

## Argument handling

- Preserve user arguments exactly.
- Do not strip `--wait` / `--background` / `--base` / `--scope` / `--model`.
- Do not add extra review instructions or rewrite intent.

## Notes vs /codex:review

- `/codex:review` uses a shared runtime (`codex-companion.mjs`) with job queue, structured schema validation, resume support.
- `/gemini:review` (v0.1.0) inlines the git diff fetch and structured prompt directly in this command — no companion runtime yet.
- Future roadmap (v0.3+): add `gemini-companion.mjs` for parity (job queue, `/gemini:status`, resume).
