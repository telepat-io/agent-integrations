# Codex Integration

## Use case

Codex supports layered AGENTS.md instructions, hook scripts, MCP integration, and skills. It is a strong target for CLI tools that need reproducible project automation.

## Required assets

- AGENTS.md section in project root.
- Hook config in .codex/hooks.json (repo) and optional ~/.codex/hooks.json (global).
- Optional skill templates and installer command.
- Optional MCP server connection model.

## Installation pattern

Recommended command shape:

```bash
toolctl codex install
toolctl codex uninstall
```

Anonymous example command:

```bash
toolctl codex install
toolctl codex uninstall
```

Observed baseline behavior:

- Adds integration guidance to AGENTS.md.
- Registers PreToolUse hook in .codex/hooks.json.
- Uses idempotent merge logic.

## Source-anchored implementation deep dive

Reference implementation anchors:

- Codex install routing and AGENTS section management in installer control flow.
- Codex hook JSON merge behavior in hook lifecycle helper functions.
- Install-path and skill-content checks in integration test coverage.
- Official behavior references for hook locations and AGENTS discovery: OpenAI Codex hooks and AGENTS docs.

What to replicate in a new CLI:

1. Keep AGENTS.md insertion marker deterministic and idempotent.
2. Merge .codex/hooks.json with existing keys preserved.
3. Document current hook coverage caveats and fail-open behavior for unsupported fields.
4. Provide a status command that reports active AGENTS and hook files.

## AGENTS.md discovery model

Codex loads guidance from global and project scopes.

Practical implications:

- Project instructions should assume global instructions may already exist.
- Narrow overrides should live closer to the working directory.
- Keep core constraints near repo root and specialized rules deeper in tree.

## Hook pattern

Codex hook configuration example:

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

Current-platform caveat:

- Codex hook interception is still evolving; do not assume complete coverage of every tool path.

## Instruction template

AGENTS.md section template:

```md
## Example integration section

- Use generated context first, raw search second.
- Keep integration artifacts up to date after code changes.
- Prefer CLI-supported query commands over ad-hoc script pipelines.
```

## MCP integration

Two common modes:

1. Codex runtime directly connected to MCP servers.
2. CLI invoked as normal tool with MCP in downstream stack.

For either mode, document exact server URL or stdio command and tool list.

## Security notes

- Require approval for high-risk remote MCP calls.
- Log sensitive action requests before execution.
- Keep allowed_tools minimal for MCP sources with large tool catalogs.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Hook not discovered | codex hooks feature disabled or wrong file path | Inspect config feature flags and .codex/hooks.json path | Enable hooks feature and place file in repo or home scope |
| Wrong AGENTS guidance | Override file precedence in parent directory | Ask Codex to list active instruction sources | Move or remove higher-precedence override file |
| Hook runs but does not enforce | Unsupported event field treated fail-open | Compare output against current Codex hook docs | Restrict logic to currently supported decision fields |
| Duplicate hook effects | Multiple hooks files match same event | Audit home and repo hook files | Consolidate to one policy owner per event |

## Validation checklist

- AGENTS.md insertion is idempotent.
- Hooks merge preserves unrelated entries.
- Uninstall removes only tool-owned entries.
- Docs reflect current hook limitations, not idealized behavior.
