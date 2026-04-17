# Codex Integration

## Use case

Codex is a strong integration target when you need all four customization surfaces together:

- instructions (`AGENTS.md`),
- lifecycle automation (hooks),
- external tool access (MCP), and
- reusable skill/plugin packaging.

## Capability support status

| Capability | Status | Primary files |
| --- | --- | --- |
| Skills | Supported | `.agents/skills/*/SKILL.md`, `~/.agents/skills/*/SKILL.md` |
| Hooks | Supported (feature-flagged, evolving) | `.codex/hooks.json`, `~/.codex/hooks.json` |
| MCP servers | Supported | `.codex/config.toml`, `~/.codex/config.toml` |
| Plugins | Supported | `.codex-plugin/plugin.json`, `.agents/plugins/marketplace.json`, `~/.agents/plugins/marketplace.json` |

## Required files and directories

Use this as a concrete starter layout:

```text
repo-root/
  AGENTS.md
  .agents/
    skills/
      release-helper/
        SKILL.md
    plugins/
      marketplace.json
  .codex/
    config.toml
    hooks.json
    hooks/
      pre_tool_use_policy.py
      post_tool_use_review.py
```

Optional global equivalents:

- `~/.agents/skills/`
- `~/.agents/plugins/marketplace.json`
- `~/.codex/hooks.json`
- `~/.codex/config.toml`

## Installation pattern

Recommended command surface in your CLI:

```bash
toolctl codex install
toolctl codex uninstall
toolctl codex status
```

Expected install behavior:

1. Insert or update a bounded `AGENTS.md` section (idempotent marker).
2. Merge hook entries into `.codex/hooks.json` without dropping unrelated hooks.
3. Optionally scaffold a starter skill in `.agents/skills/<skill-name>/SKILL.md`.
4. Optionally append MCP server config in `.codex/config.toml`.
5. Never overwrite user-owned plugin marketplace entries without explicit opt-in.

## Skills setup

### Where to place skills

- Repo-local: `.agents/skills/<skill-name>/SKILL.md`
- Global: `~/.agents/skills/<skill-name>/SKILL.md`

Codex scans repo paths from your current directory up to repo root and also reads user/admin/system locations.

### Minimal skill example

`.agents/skills/git-release/SKILL.md`:

```md
---
name: git-release
description: Create release notes and version bumps for tagged releases.
---

Use this skill when preparing a release.

1. Collect merged PR titles since the last tag.
2. Propose a semver bump and rationale.
3. Provide a copy-ready release command.
```

Optional metadata file for Codex app behavior:

`agents/openai.yaml`:

```yaml
interface:
  display_name: "Git Release"
  short_description: "Release workflow helper"
policy:
  allow_implicit_invocation: true
```

## Hooks setup

### Feature flag

Codex hooks are experimental and require enabling hooks in Codex config:

```toml
[features]
codex_hooks = true
```

### Hook locations

- Project: `<repo>/.codex/hooks.json`
- User: `~/.codex/hooks.json`

Codex loads matching hooks from all discovered files. Higher precedence does not replace lower-level hooks.

### Hook config example

`.codex/hooks.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "/usr/bin/python3 \"$(git rev-parse --show-toplevel)/.codex/hooks/pre_tool_use_policy.py\"",
            "statusMessage": "Checking command policy",
            "timeout": 30
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/usr/bin/python3 \"$(git rev-parse --show-toplevel)/.codex/hooks/stop_continue.py\""
          }
        ]
      }
    ]
  }
}
```

### Hook caveats you should document

- Current interception coverage is strongest for Bash-oriented events (`PreToolUse`, `PostToolUse`).
- Some parsed output fields are not fully enforced yet and can fail open.
- Matching hooks may run concurrently.

## MCP server setup

Codex supports MCP via CLI commands or direct config.

### CLI-driven setup

```bash
codex mcp add context7 -- npx -y @upstash/context7-mcp
codex mcp --help
```

### Config-driven setup

Project or global config:

- `.codex/config.toml`
- `~/.codex/config.toml`

Example:

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
enabled = true
tool_timeout_sec = 60

[mcp_servers.figma]
url = "https://mcp.figma.com/mcp"
bearer_token_env_var = "FIGMA_OAUTH_TOKEN"
enabled = false
```

Useful fields to expose in docs:

- `enabled_tools` and `disabled_tools` for per-server tool filtering.
- `startup_timeout_sec` and `tool_timeout_sec`.
- OAuth login flow via `codex mcp login <server-name>`.

## Plugin setup and plugin build

Codex plugins are reusable bundles that can include skills, MCP config, and app mappings.

### Minimal plugin structure

```text
my-plugin/
  .codex-plugin/
    plugin.json
  skills/
    hello/
      SKILL.md
  .mcp.json
  .app.json
  assets/
```

`.codex-plugin/plugin.json`:

```json
{
  "name": "my-first-plugin",
  "version": "1.0.0",
  "description": "Reusable greeting workflow",
  "skills": "./skills/",
  "mcpServers": "./.mcp.json"
}
```

### Marketplace file for local testing

Repo-scoped marketplace path:

- `.agents/plugins/marketplace.json`

Example:

```json
{
  "name": "local-repo",
  "interface": {
    "displayName": "Local Repo Plugins"
  },
  "plugins": [
    {
      "name": "my-first-plugin",
      "source": {
        "source": "local",
        "path": "./plugins/my-first-plugin"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Productivity"
    }
  ]
}
```

### Build workflow guidance

1. Scaffold with `$plugin-creator` when possible.
2. Keep plugin paths relative and `./`-prefixed.
3. Restart Codex after marketplace or plugin changes.
4. Validate plugin `name` stability and semantic `version` changes.

## Verification checklist

1. Run `codex mcp --help` and confirm CLI MCP commands are available.
2. Trigger `/skills` or skill mention and confirm your skill appears.
3. Confirm hooks are loaded by running a known Bash command and checking hook side effects/logs.
4. Confirm plugin appears in the plugin directory after marketplace update and restart.
5. Re-run `toolctl codex install` and verify no duplicate sections or duplicate hook entries.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Hook file exists but never runs | Hooks feature disabled | Check `[features].codex_hooks` in config | Enable feature and restart Codex |
| Hook runs inconsistently from subfolders | Relative script path depends on launch directory | Inspect hook command path resolution | Resolve script paths from git root |
| Skill not discovered | Wrong folder or missing required frontmatter | Check `skills/<name>/SKILL.md` and `name`/`description` fields | Fix path and frontmatter, then restart if needed |
| MCP server listed but tools unavailable | Startup/auth/timeouts misconfigured | Review `config.toml` server entry and auth env vars | Correct server config, re-auth with `codex mcp login` if needed |
| Plugin not visible in marketplace | Marketplace path or `source.path` invalid | Validate `.agents/plugins/marketplace.json` and plugin folder location | Fix marketplace path entries and restart Codex |
| Plugin installed but disabled | Plugin state turned off in config | Check plugin enabled state in `~/.codex/config.toml` | Set enabled state to true and restart |

## Official references

- https://developers.openai.com/codex/skills
- https://developers.openai.com/codex/mcp
- https://developers.openai.com/codex/hooks
- https://developers.openai.com/codex/plugins
- https://developers.openai.com/codex/plugins/build
