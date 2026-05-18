---
description: Delegate task to Google Gemini (default gemini-3-pro-preview, 1M context). For diagnosis, review, contra-opinie, or document analysis where Gemini's long context or alternative reasoning helps.
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

If $ARGUMENTS is empty, ask the user what they want Gemini to do.
