# Cursor Integration

## Use case

Cursor supports all major customization layers needed for repeatable CLI-agent workflows:

- skills,
- hooks,
- MCP servers, and
- instruction/rules files.

Treat Cursor as a first-class integration target, not only a rules-file target.

## Capability support status

| Capability | Status | Primary files |
| --- | --- | --- |
| Skills | Supported | `.agents/skills/*/SKILL.md`, `.cursor/skills/*/SKILL.md`, `~/.agents/skills/*/SKILL.md`, `~/.cursor/skills/*/SKILL.md` |
| Hooks | Supported | `.cursor/hooks.json`, `~/.cursor/hooks.json` |
| MCP servers | Supported | `.cursor/mcp.json`, `~/.cursor/mcp.json` |
| Rules | Supported | `.cursor/rules/*.mdc` |

## Required files and directories

Use this starter layout:

```text
repo-root/
	.cursor/
		mcp.json
		hooks.json
		hooks/
			audit.sh
			block-network.sh
		rules/
			toolctl.mdc
		skills/
			release-helper/
				SKILL.md
	.agents/
		skills/
			integration-check/
				SKILL.md
```

## Required assets

- Rule file in `.cursor/rules/` for persistent guidance.
- Optional skill directories in `.cursor/skills/` or `.agents/skills/`.
- Hook config in `.cursor/hooks.json` plus scripts in `.cursor/hooks/`.
- MCP config in `.cursor/mcp.json` for repo-scoped servers.

## Installation pattern

Recommended command shape:

```bash
toolctl cursor install
toolctl cursor uninstall
```

Suggested extended command surface:

```bash
toolctl cursor install
toolctl cursor uninstall
toolctl cursor status
```

Observed baseline behavior:

- Creates `.cursor/rules/toolctl.mdc`.
- Optionally creates `.cursor/hooks.json` and `.cursor/mcp.json` templates.
- Uses idempotent file merge or replace logic for owned sections.

## Source-anchored implementation deep dive

Reference implementation anchors:

- Cursor install and uninstall behavior in cursor helper functions.
- Rule template with frontmatter in rule-template constants.
- Rule write, idempotency, and removal checks in integration tests.
- Official Cursor docs for skills, hooks, and mcp setup behavior.

What to replicate in a new CLI:

1. Scaffold all supported Cursor surfaces, not just rules.
2. Keep rule files concise and action-oriented.
3. Use deterministic ownership markers for generated files.
4. Add a status command that verifies hooks, skills, and mcp file presence.

## Skills setup

### Skill directories

Cursor loads skills from project and global directories, including compatibility paths.

Project-level:

- `.agents/skills/`
- `.cursor/skills/`
- `.claude/skills/`
- `.codex/skills/`

User-level:

- `~/.agents/skills/`
- `~/.cursor/skills/`
- `~/.claude/skills/`
- `~/.codex/skills/`

### Minimal skill file

`.cursor/skills/release-helper/SKILL.md`:

```md
---
name: release-helper
description: Build release notes and semantic version bump proposals.
---

Use this when preparing a tagged release.
```

Important rules:

- `name` must match parent folder name.
- `name` should be kebab-case.
- Add referenced scripts and assets with relative paths from `SKILL.md`.

## Hooks setup

### Hook locations

- Project: `<repo>/.cursor/hooks.json`
- User: `~/.cursor/hooks.json`
- Optional enterprise/team layers can also apply.

### Hook config example

`.cursor/hooks.json`:

```json
{
	"version": 1,
	"hooks": {
		"beforeShellExecution": [
			{
				"command": ".cursor/hooks/block-network.sh",
				"matcher": "curl|wget|nc",
				"timeout": 30,
				"failClosed": true
			}
		],
		"afterFileEdit": [
			{
				"command": ".cursor/hooks/audit.sh"
			}
		]
	}
}
```

Use project-relative script paths under `.cursor/hooks/` for project hooks.

### Hook runtime notes

- Hooks receive JSON on stdin and may return JSON on stdout.
- Exit code `2` blocks action for command hooks.
- Cursor reloads hook config on save; restart if state appears stale.
- Agent and Tab hook events are separate and should be documented separately.

## MCP setup

### MCP config locations

- Project: `.cursor/mcp.json`
- User: `~/.cursor/mcp.json`

### Example mcp config

`.cursor/mcp.json`:

```json
{
	"mcpServers": {
		"context7": {
			"command": "npx",
			"args": ["-y", "@upstash/context7-mcp"],
			"env": {
				"CONTEXT7_API_KEY": "${env:CONTEXT7_API_KEY}"
			}
		},
		"design-api": {
			"url": "https://api.example.com/mcp",
			"auth": {
				"CLIENT_ID": "${env:MCP_CLIENT_ID}",
				"CLIENT_SECRET": "${env:MCP_CLIENT_SECRET}"
			}
		}
	}
}
```

Document both stdio and remote servers, plus oauth handling and config interpolation.

## Rules setup

Rules still matter for persistent guidance and can complement skills/hooks/mcp.

`.cursor/rules/toolctl.mdc`:

```md
---
description: toolctl integration rules
alwaysApply: true
---

- Read generated integration context before broad refactors.
- Refresh integration artifacts after code edits.
```

## Rule template

```md
---
description: toolctl context rules
alwaysApply: true
---

- Read generated architecture summary before broad codebase answers.
- Run update command after modifying code.
```

## Best practices

- Keep each surface focused: rules for policy, skills for workflows, hooks for deterministic controls, mcp for external tools.
- Avoid duplicating the same guidance in both rules and skill bodies.
- Keep hook scripts deterministic and side-effect-aware.
- Keep mcp servers minimal and disable noisy servers by default.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Skill not discovered | Invalid `SKILL.md` frontmatter or wrong directory | Check skill path and required fields | Fix path and frontmatter; reload Cursor |
| Hook not executing | Wrong hook file location or script path | Inspect `.cursor/hooks.json` and hooks output channel | Correct path and ensure script is executable |
| Hook blocks unexpectedly | Over-broad matcher or `failClosed` on flaky script | Review matcher and script exit codes | Narrow matcher and stabilize script behavior |
| MCP tools unavailable | Server config or auth failure | Check mcp server status and auth env vars | Fix `mcp.json` and re-authenticate server |
| Rules not applied | Wrong path or invalid frontmatter | Check `.cursor/rules/*.mdc` | Fix file path/frontmatter |
| Repeated instruction noise | Duplicate always-apply rules or overlapping skills | List rules and skill descriptions | Consolidate content and narrow descriptions |

## Validation checklist

1. At least one project skill appears in Cursor settings and can be invoked.
2. Hooks run on configured events and are visible in hooks diagnostics.
3. MCP server starts and exposed tools are usable from Agent chat.
4. Rule file is idempotent across reinstall.
5. Uninstall removes only tool-owned artifacts.

## Official references

- https://cursor.com/docs/skills
- https://cursor.com/docs/hooks
- https://cursor.com/docs/mcp
