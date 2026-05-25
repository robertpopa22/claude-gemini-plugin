---
description: Delegate task to Google Gemini (default gemini-2.5-pro, 1M context). For diagnosis, review, contra-opinie, or document analysis where Gemini's long context or alternative reasoning helps. Override with -m gemini-3-pro-preview for top reasoning.
argument-hint: <task description, optionally with file paths>
---

Forward $ARGUMENTS to Gemini via the `gemini-rescue` subagent.

The subagent decides between two execution patterns based on the request:

- **Stdin pipe** (default for known files): Claude reads referenced files, pipes content to Gemini. Safe, deterministic.
- **Workspace mode** (`--approval-mode plan`): Read-only sandbox for tasks requiring filesystem exploration.

Use cases:

- **Contra-opinie pe contract juridic**: paste/cite contract + ask Gemini to find risks
- **Architecture review**: paste design doc + ask Gemini if approach is sound
- **Long-document audit**: file > 100K tokens — Gemini's 1M context handles it
- **Code review second pass**: after Claude implements, ask Gemini if anything was missed
- **Codebase exploration**: "audit X in directory Y" — workspace mode

## Currency of facts (mandatory)

When the task touches any time-sensitive domain — legal, fiscal/tax, regulatory, accounting standards, ANAF/SPV procedures, library/framework versions, API contracts, or security — the forwarded prompt MUST instruct Gemini to:

- Verify the governing facts against **current** sources via its built-in Google Search (do not rely on training knowledge, which may be 6+ months stale).
- Cite **source + date** for each key fact (e.g. "OUG X/2026, in force since …", "library vN, released …").
- Explicitly flag any claim it cannot confirm as current, rather than stating it as fact.

Legal/fiscal facts (VAT rates, deadlines, amended laws, jurisprudence, fiscal forms) change often — a stale opinion is worse than none. Same discipline applies when this work is cross-checked against `/codex:review`.

If $ARGUMENTS is empty, ask the user what they want Gemini to do.
