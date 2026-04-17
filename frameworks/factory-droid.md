# Factory Droid Integration

## Use case

Factory Droid supports three first-class integration surfaces:

- skills for reusable workflows,
- MCP for external tools and APIs, and
- hooks for deterministic lifecycle controls.

AGENTS.md can still be used, but it should complement these native surfaces.

## Capability support status

| Capability | Status | Primary files and commands |
| --- | --- | --- |
| Skills | Supported | `.factory/skills/*/SKILL.md`, `~/.factory/skills/*/SKILL.md` |
| MCP servers | Supported | `.factory/mcp.json`, `~/.factory/mcp.json`, `/mcp`, `droid mcp ...` |
| Hooks | Supported | `~/.factory/settings.json`, `/hooks`, project scripts under `.factory/hooks/` |

## Required assets

- AGENTS.md section with deterministic orchestration instructions.
- Skill templates under `.factory/skills/`.
- MCP configuration in project and/or user `mcp.json`.
- Hook definitions in settings and hook scripts for reusable checks.

## Required files and directories

```text
repo-root/
	AGENTS.md
	.factory/
		skills/
			frontend-ui-integration/
				SKILL.md
		mcp.json
		hooks/
			markdown_formatter.py
```

User scope:

- `~/.factory/skills/`
- `~/.factory/mcp.json`
- `~/.factory/settings.json`

## Installation pattern

Recommended command shape:

```bash
toolctl droid install
toolctl droid uninstall
```

Suggested extended command surface:

```bash
toolctl droid install
toolctl droid uninstall
toolctl droid status
```

Observed baseline behavior:

- AGENTS.md is supplemental guidance.
- Skills define reusable task playbooks.
- MCP and hooks provide tooling and deterministic policy execution.

## Source-anchored implementation deep dive

Reference implementation anchors:

- Droid command routing and AGENTS section injection in installer control flow.
- Task-based extraction orchestration guidance in droid skill templates.
- Install-level checks for droid skill placement and AGENTS behavior in integration tests.
- Official Factory docs for skills, mcp, and hooks.

What to replicate in a new CLI:

1. Scaffold at least one `.factory/skills/<name>/SKILL.md`.
2. Provide starter MCP config in `.factory/mcp.json`.
3. Register a sample hook and script with absolute-path-safe execution.
4. Add status checks for all three surfaces.

## Skills setup

### Where skills live

- Workspace: `<repo>/.factory/skills/`
- Personal: `~/.factory/skills/`
- Compatibility: `<repo>/.agent/skills/`

Each skill should be in its own folder, for example:

`<repo>/.factory/skills/frontend-ui-integration/SKILL.md`

### Minimal SKILL.md example

```md
---
name: summarize-diff
description: Summarize staged git diff in 3-5 bullets. Use when user asks for pending changes summary.
---

# Summarize Diff

## Instructions

1. Run `git diff --staged`.
2. Summarize behavior changes and risks.
3. Call out migrations and test impact.
```

Useful frontmatter controls:

- `user-invocable`
- `disable-model-invocation`

## MCP setup

### Quickstart via interactive manager

- Type `/mcp` in Droid.
- Select Add from Registry for popular servers.
- Authenticate if required.

### CLI setup examples

HTTP server:

```bash
droid mcp add linear https://mcp.linear.app/mcp --type http
```

Stdio server:

```bash
droid mcp add playwright "npx -y @playwright/mcp@latest"
```

Remove server:

```bash
droid mcp remove linear
```

### Config files and layering

- User: `~/.factory/mcp.json`
- Project: `.factory/mcp.json`

User config overrides project config for the same server name.

Example `.factory/mcp.json`:

```json
{
	"mcpServers": {
		"linear": {
			"type": "http",
			"url": "https://mcp.linear.app/mcp",
			"disabled": false
		},
		"playwright": {
			"type": "stdio",
			"command": "npx",
			"args": ["-y", "@playwright/mcp@latest"],
			"disabled": false
		}
	}
}
```

## Hooks setup

Hooks run deterministic shell commands at lifecycle events.

### Hook events to document

- `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `Notification`, `Stop`
- `SubagentStop`, `PreCompact`, `SessionStart`, `SessionEnd`

### Quick hook registration flow

1. Run `/hooks`.
2. Choose event (for example `PreToolUse`).
3. Add matcher (for example `Execute` or `*`).
4. Add hook command.
5. Save to user settings or project context.

Example hook config shape in `~/.factory/settings.json`:

```json
{
	"hooks": {
		"PreToolUse": [
			{
				"matcher": "Execute",
				"hooks": [
					{
						"type": "command",
						"command": "jq -r '.tool_input.command' >> ~/.factory/bash-command-log.txt"
					}
				]
			}
		]
	}
}
```

Important path rule:

- Prefer absolute paths in hook commands. If project-relative, use `$FACTORY_PROJECT_DIR`.

## Orchestration template

```text
Dispatch one Task per chunk in the same response to maximize concurrency.
Wait for all tasks.
Merge valid JSON outputs only.
Skip failed chunks and report failure ratio.
```

## Best practices

- Keep each skill narrow and outcome-focused.
- Keep MCP servers minimal and disable unneeded tools.
- Encode safety checks in hooks instead of relying on prompt wording.
- Use verification checklists inside skills for enterprise reliability.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Skill not invoked | Description too broad/weak or invocation restrictions set | Check skill frontmatter and names | Tighten description and adjust invocation flags |
| MCP server appears disconnected | Auth not completed or invalid config | Use `/mcp` status and inspect config file | Re-authenticate and fix server config |
| MCP changes affect one developer only | User-level override shadowing project config | Compare `~/.factory/mcp.json` and `.factory/mcp.json` | Remove override or align user and project entries |
| Hook script fails randomly | Relative path resolves from unexpected cwd | Inspect command path in hook config | Use absolute path or `$FACTORY_PROJECT_DIR` |
| Hook not triggering | Wrong event matcher | Check matcher and event mapping in `/hooks` | Update matcher/event and retest |
| Sequential runs where parallel expected | Workflow instructions not encoded in skill | Inspect skill steps | Add explicit parallelization instructions to skill |

## Validation checklist

1. Skill exists under `.factory/skills/` and can be invoked manually.
2. Droid can auto-invoke skill when description matches task.
3. MCP server is visible and enabled in `/mcp` with tools listed.
4. Hook command executes and produces expected side effect.
5. Project and user MCP layering behavior is documented and tested.
6. Install and uninstall remain idempotent for owned files.

## Official references

- https://docs.factory.ai/cli/configuration/skills
- https://docs.factory.ai/cli/configuration/mcp
- https://docs.factory.ai/cli/configuration/hooks-guide
