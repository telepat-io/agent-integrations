# Claude Code Integration

## Use case

Claude Code is a high-control integration target for CLI tools because it supports:

- project and user instruction files,
- rich hook lifecycle,
- MCP tool usage,
- subagent-aware workflows.

## Required assets for a CLI integration

- Project instructions section in CLAUDE.md.
- Optional user-global skill registration in ~/.claude/skills.
- Hook configuration in .claude/settings.json or .claude/settings.local.json.
- Optional MCP server registration in host environment.

## Installation pattern

Recommended command shape:

```bash
toolctl claude install
toolctl claude uninstall
```

Anonymous example command:

```bash
toolctl claude install
toolctl claude uninstall
```

Observed baseline behavior:

- Inserts an integration section into project CLAUDE.md.
- Registers PreToolUse hook in .claude/settings.json.
- Supports idempotent repeat installs.

## Source-anchored implementation deep dive

Reference implementation anchors:

- Installer entrypoint and command routing in the CLI control module.
- CLAUDE markdown section template and marker management in instruction templating logic.
- Hook registration and removal behavior in hook configuration merge logic.
- Idempotency and section preservation checks in installation integration tests.

What to replicate in a new CLI:

1. Use a stable marker header in CLAUDE.md to support deterministic add/remove.
2. Merge .claude/settings.json as structured JSON, never string concat.
3. Detect existing hook registration before adding a new entry.
4. Remove only your matcher/signature on uninstall.

## Hook pattern

Typical hook structure:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Glob|Grep",
        "hooks": [
          {
            "type": "command",
            "command": "echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"allow\"}}'"
          }
        ]
      }
    ]
  }
}
```

Best practices:

- Keep matcher narrow to reduce noise and latency.
- Emit valid JSON only on stdout.
- Use stderr for diagnostics.
- Keep enforcement hooks deterministic and fast.

## Instruction strategy

Use CLAUDE.md for persistent guidance that cannot rely on hooks alone.

Recommended section shape:

```md
## Example integration section

- Before architectural answers, inspect generated context artifacts.
- After code edits, run a rebuild/update command.
- Prefer structured context over raw grep-only exploration.
```

## MCP strategy

If your CLI exposes an MCP server, keep docs explicit:

```bash
python -m yourpkg.mcp_server
```

Then document:

- tool names,
- argument schemas,
- error semantics,
- expected result format.

## Plugin marketplace strategy

Claude Code plugin marketplaces are the canonical distribution model for reusable
skills, agents, hooks, MCP servers, and LSP integrations.

Start from the user workflow:

1. Add a marketplace.
2. Install specific plugins from that marketplace.
3. Reload plugins in the active session.
4. Update marketplace metadata or specific plugin versions over time.

Core commands:

```bash
# Add from GitHub owner/repo
claude plugin marketplace add your-org/claude-plugins

# Install a plugin from that marketplace
claude plugin install formatter@your-marketplace

# Apply changes without restarting
/reload-plugins

# Refresh marketplace listings
claude plugin marketplace update your-marketplace

# Remove marketplace (also uninstalls its plugins)
claude plugin marketplace remove your-marketplace
```

Command scope guidance:

- `--scope user`: personal defaults across all repositories.
- `--scope project`: shared team config in `.claude/settings.json`.
- `--scope local`: gitignored, per-developer repository config.

Authoritative docs:

- Discover/install: https://code.claude.com/docs/en/discover-plugins
- Marketplace creation: https://code.claude.com/docs/en/plugin-marketplaces
- Full schema/reference: https://code.claude.com/docs/en/plugins-reference

## Marketplace authoring strategy

If your CLI or team distributes plugins, document marketplace authoring explicitly.

Minimum marketplace structure:

- `.claude-plugin/marketplace.json` at repository root.
- Required keys: `name`, `owner`, `plugins`.
- Each plugin entry includes at least `name` and `source`.

Minimal example:

```json
{
  "name": "company-tools",
  "owner": {
    "name": "DevTools Team"
  },
  "plugins": [
    {
      "name": "formatter",
      "source": "./plugins/formatter"
    }
  ]
}
```

Supported marketplace add sources you should document for operators:

- GitHub shorthand: `owner/repo`
- Git URL: `https://gitlab.example.com/team/plugins.git`
- Local directory path: `./my-marketplace`
- Remote URL to marketplace JSON: `https://example.com/marketplace.json`

Validation before release:

```bash
claude plugin validate .
```

## Team marketplace operations

For organization rollout, use repository settings to predeclare trusted
marketplaces and optionally pre-enable plugins.

```json
{
  "extraKnownMarketplaces": {
    "company-tools": {
      "source": {
        "source": "github",
        "repo": "your-org/claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "formatter@company-tools": true
  }
}
```

Operational policy notes:

- Treat third-party marketplaces as high trust code execution.
- Prefer project-scoped installs for team-shared workflows.
- Use managed restrictions (`strictKnownMarketplaces`) in regulated environments.

## Concrete example: MemPalace

A real marketplace install flow from the mempalace project:

```bash
claude plugin marketplace add milla-jovovich/mempalace
claude plugin install --scope user mempalace
```

After install, initialize plugin-specific setup:

```text
/mempalace:init
```

This sequence is a good reference for docs because it separates:

- marketplace registration,
- plugin installation scope,
- post-install runtime initialization.

## Security notes

- Avoid broad allow hooks unless the project is trusted.
- Use deny policies for destructive command patterns.
- Keep secrets out of hook commands and instruction files.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Hook does not run | Wrong settings file scope or matcher mismatch | Check .claude/settings.json and matcher value | Move hook to project/local file and narrow matcher |
| Hook blocks unexpectedly | Policy script uses broad pattern | Replay sample stdin payload through script | Tighten command match and add explicit allow cases |
| JSON parsing errors | Hook script writes extra stdout text | Run script with sample payload and inspect raw stdout | Ensure only one JSON object is emitted on stdout |
| Duplicate reminders | Hook duplicated across user and project layers | Inspect /hooks menu and settings files | Remove duplicate hook entries and keep one source of truth |
| Marketplace not loading | Missing or invalid `.claude-plugin/marketplace.json` | Run `claude plugin validate .` in marketplace root | Fix JSON/schema issues and retry add |
| Plugin install fails from URL marketplace | Relative `./plugins/...` source used in URL-only marketplace | Check marketplace source type and plugin `source` fields | Use Git-based marketplace add or convert plugin source to github/url/npm |
| Private marketplace update/auth errors | Missing credential helper token for background updates | Verify `git clone` works and token env vars are present | Configure `GITHUB_TOKEN`/provider token and retry update |
| Installed plugin changes not visible | Active session has stale plugin registry | Check plugin state via `/plugin` and rerun commands | Run `/reload-plugins` after install/enable/update |
| Plugin references file outside plugin root | Marketplace install copied plugin to cache | Inspect plugin paths for `../` assumptions | Use `${CLAUDE_PLUGIN_ROOT}`/`${CLAUDE_PLUGIN_DATA}` or restructure files |

## Validation checklist

- Install command writes CLAUDE.md section once.
- Reinstall remains idempotent.
- Uninstall removes only your owned section and hook entries.
- Hook output remains valid JSON under real command input.
- Marketplace add/install/update/remove commands are documented with correct scope behavior.
- Team settings examples (`extraKnownMarketplaces`, `enabledPlugins`) are syntactically valid JSON.
- Marketplace authoring guidance includes validation (`claude plugin validate .`) before release.
