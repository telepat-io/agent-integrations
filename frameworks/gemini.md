# Gemini CLI and API Integration

## Use case

Gemini supports both direct function-calling patterns and SDK-level MCP integration. It is suitable for CLI projects that need:

- strongly typed tool calls,
- optional automatic function execution,
- coding-agent workflows with skills and docs MCP.

## Required assets

- GEMINI.md section in project root for persistent behavior guidance.
- .gemini/settings.json hook config when using CLI hooks.
- Function declaration schemas for direct API usage.
- MCP server registration details when tooling is externalized.

## Installation pattern

Recommended command shape:

```bash
toolctl gemini install
toolctl gemini uninstall
```

Anonymous example command:

```bash
toolctl gemini install
toolctl gemini uninstall
```

Observed baseline behavior:

- Writes GEMINI.md section.
- Registers BeforeTool hook in .gemini/settings.json.
- Removes section and hook cleanly on uninstall.

## Source-anchored implementation deep dive

Reference implementation anchors:

- GEMINI markdown install/uninstall section and hook registration logic in installer modules.
- Gemini CLI MCP and hook setup examples in companion integration documentation.
- Stop and precompact lifecycle save logic in the hook lifecycle module.
- Official function-calling, coding-agents, and MCP guidance: ai.google.dev docs.

What to replicate in a new CLI:

1. Split Gemini integration into three concerns: instruction section, hooks, and API tool schema.
2. Keep function declarations strict and small per turn.
3. Preserve function-call id mapping when manually handling responses.
4. Treat built-in SDK MCP support as capability-dependent and keep manual fallback documented.

## Gemini CLI hook example

```json
{
  "hooks": {
    "PreCompress": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "/absolute/path/to/hooks/precompact.sh"
          }
        ]
      }
    ]
  }
}
```

## Function-calling essentials

- Use explicit parameter schemas and enums.
- Keep active tool set small to improve routing accuracy.
- Preserve function-call id mappings when manually handling conversation state.

Example declaration:

```json
{
  "name": "query_context",
  "description": "Query indexed project context for architectural questions.",
  "parameters": {
    "type": "object",
    "properties": {
      "question": { "type": "string" },
      "max_items": { "type": "integer" }
    },
    "required": ["question"]
  }
}
```

## MCP with Gemini SDK

Gemini SDK can use MCP sessions directly in supported environments.

Key caveats from official docs:

- Built-in MCP support is experimental.
- Tool support is primary focus; resources/prompts may be limited.
- Manual integration remains valid and often preferred for strict control.

## Skills and docs freshness

For coding-agent quality, combine:

- docs MCP endpoint for up-to-date API references,
- reusable skills (for example gemini-api-dev patterns).

## Security notes

- Validate function-call outputs before executing side effects.
- Use explicit approval policies for sensitive tools.
- Avoid over-broad hook matchers in shared environments.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Model answers without tools | Tool set too broad or schema too weak | Inspect declaration quality and active tool count | Reduce tool surface and improve parameter descriptions |
| Function result not applied | Missing function id mapping in manual flow | Log function_call id and function_response id | Return exact matching id in function response |
| MCP tools unavailable | Server registration or session init failed | Run gemini mcp list and test server directly | Re-register MCP server and validate startup command |
| Hook not triggering at compaction | Wrong hook section or matcher in settings | Inspect ~/.gemini/settings.json | Move hook to correct lifecycle event and matcher |

## Validation checklist

- GEMINI.md install idempotent.
- Hook registration idempotent.
- Function schemas validated against runtime constraints.
- MCP and skills verification commands documented for users.
