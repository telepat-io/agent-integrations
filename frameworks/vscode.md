# VS Code Integration

## Use case

VS Code is a multi-surface integration target. You can combine:

- Agent Skills,
- Hooks,
- MCP server hosting, and
- Agent Plugins

in one repo without building a custom VS Code extension.

## Capability support status

| Capability | Status | Primary files |
| --- | --- | --- |
| Skills | Supported | `.github/skills/`, `.claude/skills/`, `.agents/skills/`, `~/.copilot/skills/` |
| Hooks | Supported (Preview) | `.github/hooks/*.json`, `~/.copilot/hooks`, `.claude/settings.json` |
| MCP servers | Supported | `.vscode/mcp.json` (workspace), user profile `mcp.json` |
| Agent plugins | Supported (Preview) | `plugin.json`, `hooks.json`, `.mcp.json`, plugin skill folders |

## File map for repo setup

```text
repo-root/
  .github/
    skills/
      release-helper/
        SKILL.md
    hooks/
      format.json
      pretool-policy.json
  .vscode/
    mcp.json
```

User-scope equivalents:

- `~/.copilot/skills/`
- `~/.copilot/hooks`
- user profile `mcp.json` (via `MCP: Open User Configuration`)

## Agent Skills setup

### Supported skill locations

Project scope:

- `.github/skills/`
- `.claude/skills/`
- `.agents/skills/`

User scope:

- `~/.copilot/skills/`
- `~/.claude/skills/`
- `~/.agents/skills/`

### Minimal `SKILL.md`

`.github/skills/release-helper/SKILL.md`:

```md
---
name: release-helper
description: Prepare release notes and version bump proposals.
---

1. Collect merged PR summaries since the last tag.
2. Propose semantic version bump.
3. Provide release command draft.
```

Naming rules to keep explicit in docs:

- `name` must be kebab-case.
- `name` must match parent folder name.
- Invalid names can fail silently.

## Hooks setup

### Recommended workspace hook file

`.github/hooks/format.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "type": "command",
        "command": "npx prettier --write \"$TOOL_INPUT_FILE_PATH\"",
        "timeout": 30
      }
    ]
  }
}
```

### Hook locations and precedence

- Workspace hooks: `.github/hooks/*.json`
- User hooks: `~/.copilot/hooks` and Claude-compatible files
- Workspace hooks take precedence over user hooks for the same event type.

### Hook behavior notes

- VS Code uses JSON over stdin/stdout for hook I/O.
- Exit code `2` blocks processing.
- Some Claude matcher syntax is parsed for compatibility; matcher behavior can differ, so script-side filtering is safer.

## MCP server setup

### Workspace MCP config

`.vscode/mcp.json`:

```json
{
  "servers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@microsoft/mcp-server-playwright"],
      "sandboxEnabled": true,
      "sandbox": {
        "filesystem": {
          "allowWrite": ["${workspaceFolder}"]
        },
        "network": {
          "allowedDomains": ["api.example.com"]
        }
      }
    }
  }
}
```

### Setup paths and commands

- Workspace file: `.vscode/mcp.json`
- User file: command palette `MCP: Open User Configuration`
- Guided setup: `MCP: Add Server`
- Management: `MCP: List Servers`, `MCP: Reset Trust`

Important operational details:

- Enable/disable state is stored separately from `mcp.json`.
- Trust prompts appear for new workspace MCP servers.
- Sandboxing currently targets macOS/Linux; Windows caveats apply.

## Agent plugins setup

Plugins can bundle skills, hooks, agents, and MCP servers.

### Plugin structure example

```text
my-plugin/
  plugin.json
  skills/
    test-runner/
      SKILL.md
  hooks.json
  scripts/
    validate.sh
  .mcp.json
```

### Minimal `plugin.json`

```json
{
  "name": "my-dev-tools",
  "description": "Reusable development workflows",
  "version": "1.0.0",
  "skills": "skills/",
  "hooks": "hooks.json",
  "mcpServers": ".mcp.json"
}
```

Install and manage plugin surfaces in VS Code through:

- Extensions search using `@agentPlugins`
- `Chat: Install Plugin From Source`
- `chat.plugins.marketplaces` and `chat.pluginLocations`

## Verification checklist

1. Skill appears in Configure Skills and is invocable from `/` menu.
2. Hook file loads and execution appears in GitHub Copilot Chat Hooks output.
3. MCP server starts and tools are listed in Configure Tools.
4. Plugin appears in Agent Plugins view with bundled skills/hooks/MCP resources visible.
5. Re-open workspace and verify behavior persists without manual reconfiguration.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Skill not visible | Invalid skill name/folder mismatch | Check `name` and folder in `SKILL.md` path | Rename folder or `name` to match |
| Hook file ignored | Wrong location or malformed JSON | Confirm file under `.github/hooks/*.json` and inspect logs | Fix path/schema and reload |
| Hook script fails with permission error | Script not executable | Check script file mode | Run `chmod +x` on script |
| MCP server fails to start | Bad command, args, or trust rejection | Run server command directly and inspect MCP output log | Fix config and trust/start server again |
| MCP config works only for one developer | Wrong scope selected | Check whether config is workspace or user scoped | Move team config into `.vscode/mcp.json` |
| Plugin installs but does not load | Invalid `plugin.json` name or file layout | Validate plugin name and manifest location | Fix manifest and reinstall plugin |

## Official references

- https://code.visualstudio.com/docs/copilot/customization/agent-skills
- https://code.visualstudio.com/docs/copilot/customization/mcp-servers
- https://code.visualstudio.com/docs/copilot/customization/hooks
- https://code.visualstudio.com/docs/copilot/customization/agent-plugins
