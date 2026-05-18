---
description: Adversarial review with Gemini — find vulnerabilities, edge cases, risks in code/contracts/architecture that Claude may have missed. Uses gemini-3.1-pro by default.
argument-hint: <file path or description of what to review>
---

Forward an adversarial review request to Gemini via the `gemini-rescue` subagent.

The subagent will compose a Gemini prompt structured for second-opinion review:

```
You are reviewing $ARGUMENTS as an adversarial second opinion to Claude's analysis.

Identify:
1. TOP 5 vulnerabilities or risks (Critic / High / Medium severity)
2. Edge cases the primary analysis may have missed
3. Specific recommendations (concrete, not generic)
4. Verdict: ship it / ship with changes / do not ship

Be terse. Cite line numbers or specific clauses where applicable. No fluff.
```

Use cases:

- Legal contract review (pattern: ZEP audit, NDA, prestare servicii cu indemnification)
- Architecture decision review (ex: Bunavestire RSTP bridge backup plan)
- Critical code path review (security, payment, auth)
- Pre-deploy production review

If $ARGUMENTS includes a file path, the subagent will pipe its content to Gemini via stdin.
