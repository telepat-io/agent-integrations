# Snippets Cookbook

This page centralizes runnable snippets used throughout the manual.

## Install integration profiles (example CLI profile (toolctl placeholder))

```bash
# Claude
toolctl claude install

# Codex
toolctl codex install

# Gemini
toolctl gemini install

# Cursor
toolctl cursor install

# OpenCode
toolctl opencode install

# OpenClaw
toolctl claw install

# Factory Droid
toolctl droid install

# Trae variants
toolctl trae install
toolctl trae-cn install
```

## OpenClaw skills and ClawHub quick commands

```bash
# Native OpenClaw skill flows (workspace-oriented)
openclaw skills search "calendar"
openclaw skills install <skill-slug>
openclaw skills update --all
openclaw skills list

# Native OpenClaw plugin flows via ClawHub packages
openclaw plugins install clawhub:<package>
openclaw plugins update --all

# ClawHub CLI flows (auth/publish/sync automation)
clawhub login
clawhub search "postgres backups"
clawhub install <skill-slug>
clawhub update --all
clawhub sync --all
```

References:

- https://docs.openclaw.ai/tools/skills
- https://docs.openclaw.ai/tools/clawhub

## Install and remove git hooks

```bash
toolctl hook install
toolctl hook status
toolctl hook uninstall
```

## Run an MCP-enabled CLI

```bash
toolctl . --mcp
```

## Run a memory MCP server

```bash
python -m your_memory_server.mcp_server
# or with explicit palace path
python -m your_memory_server.mcp_server --palace ~/.your_memory_server/palace
```

## Claude Code MCP registration

```bash
claude mcp add your_memory_server -- python -m your_memory_server.mcp_server
```

## Gemini MCP registration

```bash
gemini mcp add your_memory_server /absolute/path/to/.venv/bin/python3 -m your_memory_server.mcp_server --scope user
```

## MCPB extension packaging quickstart

```bash
# Install MCPB CLI
npm install -g @anthropic-ai/mcpb

# Initialize/author manifest
mcpb init

# Validate and pack
mcpb validate .
mcpb pack . my-extension.mcpb

# Optional signing and verification
mcpb sign my-extension.mcpb --self-signed
mcpb verify my-extension.mcpb
```

## VS Code mcp.json example

```json
{
  "servers": {
    "filesystem": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "${workspaceFolder}"]
    },
    "toolctl": {
      "type": "stdio",
      "command": "python",
      "args": ["-m", "yourpkg.mcp_server"],
      "sandboxEnabled": true
    }
  }
}
```

## Codex hooks example

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "/usr/bin/python3 .codex/hooks/pre_tool_policy.py"
          }
        ]
      }
    ]
  }
}
```

## Claude hook output example

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Destructive command blocked"
  }
}
```

## OpenCode plugin registration example

```json
{
  "plugin": [
    ".opencode/plugins/toolctl.js"
  ]
}
```

## Gemini function declaration example

```json
{
  "name": "search_project_memory",
  "description": "Search indexed project memory and return the best matching entries.",
  "parameters": {
    "type": "object",
    "properties": {
      "query": { "type": "string" },
      "limit": { "type": "integer" }
    },
    "required": ["query"]
  }
}
```

## Remote HTTP MCP server entry (JSON host)

```json
{
  "mcpServers": {
    "docs-provider": {
      "url": "https://mcp.example.com/mcp",
      "headers": {
        "CONTEXT7_API_KEY": "YOUR_API_KEY"
      }
    }
  }
}
```

## Remote HTTP MCP server entry (TOML host)

```toml
[mcp_servers.docs_provider]
url = "https://mcp.example.com/mcp"
http_headers = { CONTEXT7_API_KEY = "YOUR_API_KEY" }
```

## Append-marker instruction injection block

```markdown
<!-- docs-provider -->
Use docs-provider MCP whenever the user asks about external libraries, SDK APIs, or version-specific setup.

Workflow:
1. Resolve canonical library id.
2. Query docs using the selected id and the full user question.
3. Answer using fetched docs and code examples.
<!-- docs-provider -->
```

## Skill frontmatter trigger template

```yaml
---
name: docs-lookup
description: Use when the user asks about library APIs, framework setup, or version-specific examples.
---
```

## Minimal SKILL.md template

```markdown
---
name: diagnostics
description: Use this skill when users ask to debug integration setup, missing config, or activation failures, even if they describe symptoms instead of naming the root cause.
---

## Workflow

1. Detect framework and scope (repo, user, or system).
2. Run status checks and summarize missing artifacts.
3. Propose least-risk remediation steps.
4. Re-run status checks and report final state.
```

## Description quality examples

```yaml
# Weak
description: Helps with setup.

# Better
description: Use this skill when users are installing, repairing, or validating CLI agent integrations (skills, instructions, hooks, or MCP config), including cases where they report symptoms such as missing commands, inactive skills, or malformed config files.
```

## Trigger eval query set shape

```json
[
  {
    "query": "I installed the integration but /skills is empty and it says no templates were found under my repo.",
    "should_trigger": true
  },
  {
    "query": "Give me a high-level summary of MCP history.",
    "should_trigger": false
  }
]
```

## Eval case schema with assertions

```json
{
  "skill_name": "diagnostics",
  "evals": [
    {
      "id": 1,
      "prompt": "Our Codex setup stopped loading repo skills after a config change. Diagnose and fix it.",
      "expected_output": "A concrete diagnosis and a validated remediation path.",
      "files": ["evals/files/config.toml"],
      "assertions": [
        "Identifies the specific broken config key or path",
        "Provides a concrete remediation command or file edit",
        "Includes a post-fix verification step"
      ]
    }
  ]
}
```

## Script interface contract for skills

```bash
# Non-interactive usage
python scripts/check_integration.py --framework codex --scope repo --format json

# Help text must be complete and concise
python scripts/check_integration.py --help
```

```text
Usage: check_integration.py --framework <name> --scope <repo|user|system> [--format json|table]

Checks agent integration artifacts and returns machine-parseable results.

Options:
  --framework   Target framework profile
  --scope       Config scope
  --format      Output format (default: json)
  --verbose     Print diagnostics to stderr
```

```json
{"status":"fail","missing":[".agents/skills/diagnostics/SKILL.md"],"next_step":"run toolctl codex install"}
```

## Integration surface audit script

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Checking MCP config artifacts..."
test -f .mcp.json || test -f .cursor/mcp.json || true

echo "Checking skill artifacts..."
test -f .claude/skills/docs-lookup/SKILL.md || test -f .agents/skills/docs-lookup/SKILL.md || true

echo "Checking rule artifacts..."
test -f .claude/rules/docs-provider.md || grep -n "<!-- docs-provider -->" AGENTS.md || true

echo "Reminder: if hook callbacks are unsupported in this profile, keep fallback governance in instructions."
```
