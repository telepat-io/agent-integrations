# Trae and Trae-CN Integration

## Use case

Trae supports two core integration layers for agent workflows:

- skills for reusable capability instructions, and
- MCP servers for external tool access.

Use AGENTS.md as supplemental guidance, not as the only integration surface.

## Capability support status

| Capability | Status | Primary files |
| --- | --- | --- |
| Skills | Supported | `.trae/skills/*/SKILL.md`, `~/.trae/skills/*/SKILL.md`, optional `.agents/skills/*/SKILL.md` |
| MCP servers | Supported | `.trae/mcp.json` (project), Trae settings MCP config |
| Install links for MCP | Supported | `trae://trae.ai-ide/mcp-import?...` schema link |
| Hooks | Not documented in provided Trae sources | Treat as unsupported until vendor docs confirm |

## Required assets

- AGENTS.md section with explicit integration rules.
- Skill files in `.trae/skills/` and optional compatibility skills in `.agents/skills/`.
- MCP server definitions via Settings > MCP and optional project-level `.trae/mcp.json`.

## Required files and directories

```text
repo-root/
	AGENTS.md
	.trae/
		skills/
			code-review/
				SKILL.md
		mcp.json
		skill-config.json
	.agents/
		skills/
			release-helper/
				SKILL.md
```

Notes:

- `.trae/skills/` takes precedence over `.agents/skills/` when skill names collide.
- `skill-config.json` is created by Trae to track disabled project skills.

## Installation pattern

Recommended command shape:

```bash
toolctl trae install
toolctl trae uninstall
toolctl trae-cn install
toolctl trae-cn uninstall
```

Suggested extended command surface:

```bash
toolctl trae install
toolctl trae-cn install
toolctl trae status
```

Observed baseline behavior:

- Both trae and trae-cn should install the same file layout strategy.
- Skill and MCP scaffolding are the main value-add over AGENTS-only setup.

## Source-anchored implementation deep dive

Reference implementation anchors:

- Trae and trae-cn command paths and AGENTS install flow in installer control logic.
- Shared Trae skill template and path behavior guidance.
- Integration tests validating trae and trae-cn target path behavior.
- Official Trae docs for skills and MCP flows.

What to replicate in a new CLI:

1. Scaffold `.trae/skills/` with at least one concrete skill template.
2. Add optional `.agents/skills/` compatibility guidance and precedence notes.
3. Scaffold `.trae/mcp.json` with stdio and/or HTTP examples.
4. Add a status command that checks both skills and MCP readiness.

## Skills setup

### Skill locations

- Project skills: `.trae/skills/<skill-name>/SKILL.md`
- Global skills: `~/.trae/skills/<skill-name>/SKILL.md`
- Compatibility path: `.agents/skills/<skill-name>/SKILL.md`

### Minimal SKILL.md format

`.trae/skills/code-review/SKILL.md`:

```md
---
name: code-review
description: Review changed code for correctness, risk, and missing tests.
---

# Code Review

## Description
Review code changes and produce prioritized findings.

## When to use
Use when user asks for review or code quality feedback.

## Instructions
1. Inspect changed files first.
2. List findings by severity.
3. Include test and rollback risks.
```

### Skill management behavior

- Skills can be created in chat, manually in settings, or imported from files/zip.
- Skills can be enabled/disabled in settings.
- Project disabled state is tracked in `.trae/skill-config.json`.

## MCP setup

Trae supports stdio, SSE, and streamable HTTP MCP transports.

### Add MCP from settings

Flow:

1. Open Settings > MCP.
2. Add from Marketplace or Add Manually.
3. Paste config JSON and confirm.

### Project-level MCP config

Create `.trae/mcp.json` and enable project MCP in Settings > MCP.

Stdio example:

```json
{
	"mcpServers": {
		"github": {
			"command": "npx",
			"args": ["-y", "@modelcontextprotocol/server-github"],
			"env": {
				"GITHUB_PERSONAL_ACCESS_TOKEN": "<YOUR_TOKEN>"
			}
		}
	}
}
```

HTTP example:

```json
{
	"mcpServers": {
		"remote-api": {
			"url": "https://example.com/mcp",
			"headers": {
				"Authorization": "Bearer <TOKEN>"
			}
		}
	}
}
```

Timeout and variable notes to document:

- Stdio supports timeout values through `env` fields.
- HTTP supports timeout through `headers` fields.
- `${workspaceFolder}` variable is supported in command args.

## MCP install links

Trae supports schema-based install links:

`trae://trae.ai-ide/mcp-import?type=${TYPE}&name=${NAME}&config=${BASE64_ENCODED_CONFIG}`

Use this for one-click distribution of server configs across teams.

## Agent usage with MCP

- Built-in `Builder with MCP` includes all configured MCP servers.
- Custom agents can select specific MCP servers and tools in Tools > MCP.

## Best practices

- Keep each skill focused and specific in trigger description.
- Put project-specific operational playbooks in project skills, global defaults in global skills.
- Treat `.trae/mcp.json` as code and review it like any other config file.
- Require trusted workspaces before enabling project-level MCP.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Skill not invoked automatically | Description too vague or disabled skill | Check skill description and `.trae/skill-config.json` | Improve description and re-enable skill |
| Wrong skill version loaded | Name collision between `.trae/skills` and `.agents/skills` | Check both directories for same skill name | Rename or remove duplicate skill names |
| MCP server fails to start | Invalid JSON config or missing local runtime (npx/uvx) | Validate MCP JSON and local command availability | Fix config and install required runtime |
| Project MCP not applied | Project MCP switch disabled | Check Settings > MCP project toggle | Enable project MCP and reload |
| Install link import fails | Malformed or non-encoded config parameter | Validate schema link and URL-encoded base64 payload | Regenerate link and retry import |
| Trae-CN mismatch with Trae behavior | Only one variant updated | Verify both install targets and generated files | Re-run installs for both variants |

## Validation checklist

1. Install and uninstall are idempotent for both trae and trae-cn.
2. Project and global skills appear in Settings > Rules & Skills.
3. `.agents/skills` compatibility loading is enabled when used.
4. MCP server appears in Settings > MCP and tools are available in agent flow.
5. Project-level `.trae/mcp.json` loads when project MCP is enabled.

## Official references

- https://docs.trae.ai/ide/skills
- https://docs.trae.ai/ide/model-context-protocol?_lang=en
- https://docs.trae.ai/ide/add-mcp-servers?_lang=en
- https://docs.trae.ai/ide/mcp-server-install-links?_lang=en
- https://docs.trae.ai/ide/use-mcp-servers-in-agents?_lang=en
