# DEPLOY — applying plugin edits

> **Why this file exists:** Claude Code does **not** run this plugin from the repo. At install time it
> copies the plugin files into a versioned cache dir. Editing the repo has **no effect** until the cache
> is re-synced **and** Claude Code is restarted. This doc is the deploy step so we don't rediscover it.

## How it loads (this machine)

- Marketplace `geseidl-plugins` → source `directory` = `D:\github\claude-gemini-plugin`
  (marketplace metadata is read live from here).
- Plugin payload (`commands/`, `agents/`, `skills/`, …) is a **snapshot copy** at:
  `C:\Users\<user>\.claude\plugins\cache\geseidl-plugins\gemini\<version>\`
- Config: `~\.claude\plugins\installed_plugins.json` (records `installPath` + `version`),
  `~\.claude\plugins\known_marketplaces.json` (records the directory source).

So: **repo edits → cache snapshot → restart** before they take effect.

## Deploy after editing the plugin

1. Edit files in `D:\github\claude-gemini-plugin\` (repo = source of truth, commit them).
2. Re-sync the cache snapshot (mirror repo → cache, exclude `.git`):

   ```powershell
   $repo  = "D:\github\claude-gemini-plugin"
   $ver   = (Get-Content "$repo\.claude-plugin\plugin.json" | ConvertFrom-Json).version
   $cache = "$env:USERPROFILE\.claude\plugins\cache\geseidl-plugins\gemini\$ver"
   robocopy $repo $cache /E /XD .git | Out-Null
   if ($LASTEXITCODE -ge 8) { throw "robocopy failed ($LASTEXITCODE)" } else { "synced -> $cache" }
   ```

   - `robocopy` exit codes 0–7 = success (1 = files copied); only `>=8` is a real error.
   - If you bump `version` in `.claude-plugin/plugin.json`, the cache path changes — the official
     `/plugin` reinstall would create a new versioned dir. The manual mirror above writes into the
     **current** `version` dir, so no bump is required for a content-only change.
3. **Restart Claude Code** (close + reopen the session). Plugin commands/agents/skills are loaded at
   startup; a running session will not pick up changes mid-session.
4. Verify: in a fresh session run the changed command (e.g. `/gemini:review`) or
   `grep "Currency of facts" "$cache\commands\rescue.md"`.

## Alternative (official, verify the menu)

`/plugin` → manage marketplaces → update `geseidl-plugins` → reinstall `gemini`. This re-copies from the
directory source. Prefer the manual mirror above for a quick content-only deploy; it is deterministic and
needs no CLI flags.

## Note on workstation independence

The `directory` source (`D:\github\...`) is local to this machine. On another workstation, install from the
GitHub marketplace (`robertpopa22/claude-gemini-plugin`) per `README.md`, then the cache lives under that
machine's `~\.claude\plugins\cache\`. The deploy step (re-sync + restart) is the same.
