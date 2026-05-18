---
name: gemini-cli-runtime
description: Internal helper contract for calling the Gemini CLI from Claude Code. Defines auth check, call patterns, error handling, model selection, and stdin piping for large context.
---

# Gemini CLI Runtime Contract

This skill documents how the `gemini-rescue` subagent invokes the Gemini CLI.

## Prerequisites

1. **CLI installed**: `npm install -g @google/gemini-cli` (v0.42.0+, released weekly Tuesdays)
2. **OAuth login**: `gemini auth login` (one-time, browser flow, token persisted in `~/.gemini/`)
3. **No API key needed** for free tier — OAuth via Google account uses Gemini Code Assist quota (60 req/min, 1000/day at time of writing)

## Valid model slugs (verified 2026-05-18 with gemini-cli v0.42.0)

CLI `-m` flag accepts these values:

| Slug | Class | Notes |
|------|-------|-------|
| `auto` | Auto | Default Gemini 3 routing (recommended starting point) |
| `gemini-3` | Auto | Auto Gemini 3 routing |
| `gemini-2.5` | Auto | Auto Gemini 2.5 routing |
| `gemini-3-pro-preview` | Pro | **Recommended default** — top reasoning, 1M context |
| `gemini-3-flash-preview` | Flash | Fast, lower cost |
| `gemini-2.5-pro` | Pro | Stable GA |
| `gemini-2.5-flash` | Flash | Stable GA fast |

**Notes**:
- `gemini-3.1-pro` is the **release name**, NOT a valid CLI slug — using it gives 404.
- Model availability depends on auth tier — OAuth (Code Assist) free tier may not include all preview models. If `gemini-3-pro-preview` fails with 404, fall back to `auto` or `gemini-2.5-pro`.
- Pricing only applies to paid API key auth. OAuth has quota limits (60 req/min, 1000/day) but no per-token billing.

## Sandbox & approval modes

Gemini CLI runs in a sandbox by default — without explicit permission, it cannot read files outside the workspace, run shell commands, or edit. For a **second-opinion review**, this is a problem: the model needs content to opine on.

Two strategies:

### Strategy 1 — Stdin pipe (PREFERRED for known content)

Claude Code (the caller) reads files with its own Read tool, then pipes content to Gemini's stdin:

```bash
cat /path/to/file1.py /path/to/file2.md | gemini -m gemini-3-pro-preview -p "<prompt>"
```

Gemini sees exactly the piped content. No sandbox config needed. **Safe default.**

### Strategy 2 — Workspace read-only mode

When Gemini must navigate a codebase or directory, run with `--approval-mode plan` (read-only):

```bash
gemini -m gemini-3-pro-preview --approval-mode plan -p "Review architecture of D:\github\my-project"
```

`--approval-mode` values:
- `default` — prompt for each action (interactive only; useless headless)
- `plan` — read-only (read files, list dirs; no edits, no shell)
- `auto_edit` — auto-approve edits (use ONLY when user explicitly wants Gemini to make changes)
- `yolo` — auto-approve everything (DO NOT use; security risk)

### When to use which

| Task | Strategy |
|------|----------|
| Review one or two files Claude already knows | Strategy 1 (stdin pipe) |
| Audit a contract paste | Strategy 1 (stdin pipe of the contract text) |
| "Find issues in `/path/to/project/`" | Strategy 2 (`--approval-mode plan`) |
| "Refactor X across the repo" | Strategy 2 (`--approval-mode auto_edit`, only with explicit user OK) |

## Call Patterns

### Basic prompt only

```bash
gemini -m gemini-3-pro-preview -p "Explain ..."
```

### Single-file review (stdin pipe)

```bash
cat /path/to/document.md | gemini -m gemini-3-pro-preview -p "Identify top 5 risks"
```

### Multi-file review (stdin pipe)

```bash
cat file1.py file2.py file3.py | gemini -m gemini-3-pro-preview -p "Audit for race conditions"
```

### Codebase exploration (read-only workspace)

```bash
gemini -m gemini-3-pro-preview --approval-mode plan -p "Audit auth flow in this Next.js app"
```

## Error Handling

- **OAuth error** (`Authentication required` or similar): instruct user to run `gemini auth login` once, then retry. Do NOT loop attempts.
- **Quota exceeded** (`429` or `quota` keyword): report verbatim, suggest retry in ~1 min (60 req/min limit).
- **Network error**: report verbatim, do not retry automatically.
- **Model not found**: fall back to `gemini-3-pro` (GA) only if explicit user instruction; otherwise report error.

## Output Capture

- Return stdout exactly as-is. No reformatting, no truncation, no summarization.
- If output is empty (rare), return `[Gemini returned empty response]` so the caller knows the call succeeded but yielded nothing.
- For very long outputs (>5000 lines), the caller may decide to truncate at the display layer — the wrapper itself does not truncate.

## Anti-patterns

- **Do NOT** add `--debug` or verbose flags unless user explicitly requests.
- **Do NOT** chain multiple gemini calls in one Bash invocation (e.g. `gemini ... && gemini ...`) — return after the first.
- **Do NOT** use `--yolo` (auto-approve everything including shell). It bypasses safety checks. Use `--approval-mode plan` or stdin pipe instead.
- **Do NOT** parse Gemini's output and re-format. Trust the model's chosen format.
- **Do NOT** call Gemini in interactive mode (without `-p`). Always use `-p "<prompt>"` for headless one-shot.

## Cold start latency

First call after install can take 30-60s (model warmup). Subsequent calls in the same session are fast. If the first call hangs, give it up to 90s before assuming failure.

## Cost & Rate Limits

OAuth free tier:
- 60 req/min
- 1000 req/day
- Sufficient for ~20-30 adversarial reviews per workday

If exceeding free tier, user must configure paid API key via env var or settings.
