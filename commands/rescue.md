---
description: Delegate task to Google Gemini (default gemini-3.1-pro, 1M context). For diagnosis, review, contra-opinie, or document analysis where Gemini's long context or alternative reasoning helps.
argument-hint: <task description>
---

Forward $ARGUMENTS to Gemini via the `gemini-rescue` subagent.

The subagent will invoke `gemini -m gemini-3.1-pro -p "<task>"` and return Gemini's response verbatim.

Use cases:

- **Contra-opinie pe contract juridic**: paste contract + ask Gemini to find risks
- **Architecture review**: paste design doc + ask Gemini if approach is sound
- **Long-document audit**: stdin pipe a file > 100K tokens (use `cat <file> | gemini -p "review"` pattern)
- **Code review second pass**: after Claude implements, ask Gemini if anything was missed

If $ARGUMENTS is empty, ask the user what they want Gemini to do.
