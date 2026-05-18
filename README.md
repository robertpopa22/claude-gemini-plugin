# claude-gemini-plugin

> Use Google Gemini as a second-opinion engine inside Claude Code.

Forward tasks to Gemini for adversarial review, legal contract analysis, architecture decisions, or any case where a different model family helps catch what Claude missed. Inspired by the [`openai-codex`](https://github.com/anthropics/claude-code) plugin pattern.

## Features

- **`@gemini-rescue` subagent** — thin forwarding wrapper, returns Gemini output verbatim
- **`/gemini:rescue <task>`** — delegate any task to Gemini (default `gemini-3.1-pro`, 1M token context)
- **`/gemini:review <path>`** — adversarial review for contracts, code, architecture
- **OAuth-based auth** — uses your Google account (Gemini Code Assist free tier: 60 req/min, 1000/day)

## Install

### 1. Install the Gemini CLI

```bash
npm install -g @google/gemini-cli
gemini auth login   # one-time browser OAuth
```

Verify: `gemini --version` should return `0.42.0` or later.

### 2. Install this plugin

Clone or symlink into your Claude Code plugins cache:

```bash
# Recommended: clone into your plugins directory
git clone https://github.com/robertpopa22/claude-gemini-plugin \
  ~/.claude/plugins/cache/robertpopa22-claude-gemini/gemini/0.1.0
```

Restart Claude Code. The `@gemini-rescue` agent and `/gemini:*` commands will be available.

## Usage

### Adversarial contract review

```
/gemini:review D:\contracts\NDA-vendor-X.docx
```

Gemini will return top 5 risks, edge cases, recommendations, and a ship/no-ship verdict.

### Long-document analysis (1M context)

```
/gemini:rescue Citeste docs/architecture-2026.md (450K tokens) si gaseste inconsistente
```

Gemini sees the full document — Claude with smaller context would truncate.

### Direct subagent use (from main thread)

When Claude detects a task suited to Gemini, it can spawn the subagent proactively:

> *Claude*: "This contract has unusual indemnification clauses. Let me get a second opinion from Gemini before you sign."

## Configuration

| Setting | Default | How to change |
|---------|---------|---------------|
| Model | `gemini-3.1-pro` | Pass `-m gemini-3-pro` (or any valid model name) in your prompt |
| Auth | OAuth (free tier) | `gemini auth login` |
| Context size | 1M tokens | Model-determined |

## Why Gemini + Claude Together?

Different model families have different blind spots. A second opinion from a different lineage catches errors that single-model self-review misses. We've used this pattern internally at Geseidl for:

- **Legal contracts** — Codex + Gemini contra-opinie identified 5 vulnerabilities on a contract Claude alone marked as safe
- **Architecture reviews** — Gemini's 1M context lets us review entire codebases that exceed Claude's working window
- **Production incident postmortems** — different model = different angle on root cause analysis

## Compatibility

- Claude Code 2.0+
- Node.js 18+ (for `@google/gemini-cli`)
- macOS / Linux / Windows (PowerShell or MSYS2 bash)

## Status

**v0.1.0** — initial release. Foreground only, no background job queue, no `--resume-last`. Sufficient for review/rescue workflows.

Roadmap:
- v0.2 — `/gemini:status` + structured output schemas
- v0.3 — background companion (`gemini-companion.mjs` for long-running tasks)
- v0.4 — lifecycle hooks + MCP exposure

## Maintained by

<a href="https://geseidl.ro/servicii-it"><img src="https://geseidl.ro/assets/icons/logo-green.png" alt="Geseidl Consulting Group" height="40"></a>

This plugin is part of our **multi-model workflow** at Geseidl IT Solutions — we use Claude for primary development, Codex for write-capable rescue, and Gemini for second-opinion adversarial review. Open-sourcing the integration because great tools should be shared.

Maintained by [Geseidl IT Solutions](https://geseidl.ro/servicii-it), part of [Geseidl Consulting Group](https://geseidl.ro).

## License

MIT — see [LICENSE](LICENSE).

---

<p align="center">
  <a href="https://make-it-count.ro">
    <img src="https://geseidl.ro/assets/icons/makeitcount-amprenta-gold.png" alt="makeitcount" height="60">
  </a>
  <br>
  <sub><em>We believe great tools should be shared. Every contribution counts.</em></sub>
  <br>
  <sub><a href="https://make-it-count.ro">make-it-count.ro</a></sub>
</p>
