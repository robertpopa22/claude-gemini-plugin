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

## Models (as of 2026-05-18)

| Model | Status | Context | Output | Pricing (input/output per 1M) |
|-------|--------|---------|--------|-------------------------------|
| `gemini-3.1-pro` | Preview (released 2026-02-19) | 1M | 65K | $2 / $12 (under 200K), $4 / $18 (over) |
| `gemini-3-pro` | GA | 1M | 65K | $2 / $12 |
| `gemini-3-pro-preview` | **DEPRECATED** 2026-03-09 | — | — | — |

**Default**: `gemini-3.1-pro` for top reasoning. Use `gemini-3-pro` for stable GA workflows.

## Call Patterns

### Basic invocation

```bash
gemini -m gemini-3.1-pro -p "<prompt>"
```

### Large context (file > 100K tokens)

Pipe via stdin:

```bash
cat /path/to/large/document.md | gemini -m gemini-3.1-pro -p "Review this document for: ..."
```

### Multi-file review

Concatenate first:

```bash
cat file1.py file2.py file3.py | gemini -m gemini-3.1-pro -p "Audit for race conditions"
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
- **Do NOT** use Gemini's built-in tools (`--enable-shell`, `--enable-fs`) — those overlap with Claude Code's own tools and create confusion.
- **Do NOT** parse Gemini's output and re-format. Trust the model's chosen format.

## Cost & Rate Limits

OAuth free tier:
- 60 req/min
- 1000 req/day
- Sufficient for ~20-30 adversarial reviews per workday

If exceeding free tier, user must configure paid API key via env var or settings.
