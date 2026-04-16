# VS Code as MCP Host

## Use case

VS Code can host local and remote MCP servers and expose their tools in Copilot Chat and related agent experiences. For CLI projects, this enables first-class developer workflows without custom IDE extensions.

## Core config file

Use mcp.json in one of two scopes:

- Workspace scope: .vscode/mcp.json (committable and team-shareable).
- User scope: profile-level mcp.json (local, cross-workspace).

Example:

```json
{
  "servers": {
    "toolctl": {
      "type": "stdio",
      "command": "python",
      "args": ["-m", "yourpkg.mcp_server"],
      "sandboxEnabled": true,
      "sandbox": {
        "filesystem": {
          "allowWrite": ["${workspaceFolder}"]
        }
      }
    }
  }
}
```

## Best-practice host integration

- Ship a validated example mcp.json snippet in your docs.
- Provide a doctor command to verify server startup outside VS Code first.
- Keep stdio server logs on stderr only.
- Include trust and sandbox guidance in quickstart docs.

## Source-anchored implementation deep dive

Reference implementation anchors:

- Sample CLI MCP server entrypoint and query tools in MCP server and CLI routing modules.
- A companion state MCP server request handling and tool dispatch module.
- Official host behavior for mcp.json scopes, trust, and sandboxing: VS Code MCP docs.

What to replicate in a new CLI:

1. Validate MCP server startup from terminal before IDE integration.
2. Provide copy-safe mcp.json snippets for workspace and user scopes.
3. Keep stdio output protocol-clean and use stderr for logs.
4. Add trust and sandbox recommendations in quickstart docs.

## Operational behavior

- VS Code discovers tools after server starts.
- Server enable/disable state is managed separately from raw mcp.json config.
- Trust prompts appear for new workspace servers unless bypassed by policy.

## Security model

- Treat local stdio servers as code execution surfaces.
- Recommend sandboxing where available.
- Use input variables or env files for secrets, never literal credentials in mcp.json.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Server fails to start in VS Code | Wrong command/interpreter in mcp.json | Run the same command directly in terminal | Fix command path and args in mcp.json |
| Server starts but no tools shown | tools/list fails or returns empty set | Check MCP output logs and direct tools/list test | Fix server init/list implementation |
| Config works on one machine only | Wrong scope used (user vs workspace) | Inspect mcp.json location | Move to correct scope and commit workspace config |
| Security warning on startup | Trust not granted for workspace server | Check MCP trust state and prompts | Review config and explicitly trust known server |

## Validation checklist

- Stdio server starts from terminal before VS Code integration.
- tools/list returns non-empty set.
- mcp.json schema valid and checked into expected scope.
- sandbox rules documented for macOS/Linux and caveat for Windows.
