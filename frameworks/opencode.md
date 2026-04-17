# OpenCode Integration

## Use case

OpenCode supports multiple integration surfaces for CLI projects:

- skills via SKILL.md,
- MCP servers via opencode.json,
- custom tools in .opencode/tools/, and
- optional plugin and rules layering.

Use all applicable surfaces rather than treating OpenCode as plugin-only.

## Capability support status

| Capability | Status | Primary files |
| --- | --- | --- |
| Skills | Supported | `.opencode/skills/*/SKILL.md`, `~/.config/opencode/skills/*/SKILL.md`, compatibility skill paths |
| MCP servers | Supported | `opencode.json` under `mcp` |
| Custom tools | Supported | `.opencode/tools/*.{ts,js}` |
| Plugin registration | Supported | `.opencode/plugins/*`, `opencode.json` |

## Required assets

- AGENTS.md section in project root.
- OpenCode config file: opencode.json.
- Skill files in .opencode/skills/ or compatibility skill folders.
- Optional custom tools in .opencode/tools/.
- Optional plugin file in .opencode/plugins/ plus config registration.

## Required files and directories

```text
repo-root/
  AGENTS.md
  opencode.json
  .opencode/
    skills/
      git-release/
        SKILL.md
    tools/
      database.ts
      math.ts
    plugins/
      toolctl.js
```

## Installation pattern

Recommended command shape:

```bash
toolctl opencode install
toolctl opencode uninstall
```

Suggested extended command surface:

```bash
toolctl opencode install
toolctl opencode uninstall
toolctl opencode status
```

Expected install behavior:

- Writes AGENTS.md guidance section.
- Writes or merges opencode.json for mcp and tool permissions.
- Optionally writes .opencode/skills/<name>/SKILL.md.
- Optionally writes .opencode/tools/<name>.ts.
- Optionally writes .opencode/plugins/toolctl.js and registers it.

## Source-anchored implementation deep dive

Reference implementation anchors:

- OpenCode install and uninstall command routing and AGENTS handling in installer control flow.
- Plugin template source and registration merge behavior in OpenCode plugin helpers.
- Plugin write, merge, and cleanup checks in integration tests.
- Official OpenCode docs for skills, mcp servers, and custom tools.

What to replicate in a new CLI:

1. Treat opencode.json as the source of truth for mcp and tool policy.
2. Parse existing opencode.json safely and preserve unrelated keys.
3. Scaffold at least one real skill and one custom tool example.
4. Keep plugin registration optional and explicitly owned.

## Skills setup

### Skill file locations

Project-local:

- .opencode/skills/<name>/SKILL.md
- .claude/skills/<name>/SKILL.md
- .agents/skills/<name>/SKILL.md

Global:

- ~/.config/opencode/skills/<name>/SKILL.md
- ~/.claude/skills/<name>/SKILL.md
- ~/.agents/skills/<name>/SKILL.md

### Minimal skill example

.opencode/skills/git-release/SKILL.md:

```md
---
name: git-release
description: Create consistent release notes and version bumps.
---

Use this when preparing a tagged release.
```

Name constraints to enforce in docs:

- lowercase alphanumeric plus single hyphens,
- must match containing directory name,
- unique names across loaded locations.

## MCP server setup

OpenCode configures MCP servers in opencode.json under mcp.

### Local server example

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "context7": {
      "type": "local",
      "command": ["npx", "-y", "@upstash/context7-mcp"],
      "enabled": true,
      "environment": {
        "CONTEXT7_API_KEY": "{env:CONTEXT7_API_KEY}"
      }
    }
  }
}
```

### Remote server example with oauth

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "sentry": {
      "type": "remote",
      "url": "https://mcp.sentry.dev/mcp",
      "oauth": {},
      "enabled": true
    }
  }
}
```

### Useful commands to document

```bash
opencode mcp list
opencode mcp auth sentry
opencode mcp auth list
opencode mcp debug sentry
opencode mcp logout sentry
```

## Custom tools setup

### Tool locations

- Project: .opencode/tools/
- Global: ~/.config/opencode/tools/

### Single tool example

.opencode/tools/database.ts:

```ts
import { tool } from "@opencode-ai/plugin";

export default tool({
  description: "Query the project database",
  args: {
    query: tool.schema.string().describe("SQL query to execute")
  },
  async execute(args) {
    return `Executed query: ${args.query}`;
  }
});
```

### Multiple exports example

.opencode/tools/math.ts can export multiple tools. Each export maps to <filename>_<exportname>.

## Plugin pattern

Plugins are optional in OpenCode but useful for reusable pre-tool behavior.

Example plugin behavior:

```js
export const ToolPlugin = async ({ directory }) => {
  return {
    "tool.execute.before": async (input, output) => {
      if (input.tool === "bash") {
        output.args.command = "echo '[context hint]' && " + output.args.command;
      }
    }
  };
};
```

## Configuration snippet

opencode.json:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "context7": {
      "type": "remote",
      "url": "https://mcp.context7.com/mcp",
      "enabled": true
    }
  },
  "plugin": [
    ".opencode/plugins/toolctl.js"
  ]
}
```

## Safety notes

- Validate opencode.json before write to avoid silent plugin failure.
- Keep plugin logic small and deterministic.
- Avoid output mutation that can change command meaning unexpectedly.
- Keep MCP usage scoped to required servers and disable noisy servers by default.
- Prefer explicit permissions for skill and tool access in sensitive environments.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Skill not visible | Invalid SKILL.md name or path | Check skill path, frontmatter, and uniqueness | Fix naming and path, then reload |
| MCP tools unavailable | Server disabled or auth failed | Run opencode mcp list and opencode mcp debug <name> | Enable server and complete auth flow |
| OAuth flow fails | Wrong oauth config or provider setup | Run opencode mcp auth list and debug output | Fix oauth settings and re-authenticate |
| Custom tool not callable | Tool file path or export shape invalid | Check .opencode/tools/ and exported tool objects | Fix file and export and restart session |
| Plugin not loaded | Missing plugin entry in opencode.json | Inspect plugin array and file existence | Re-register plugin path and restart host |
| Config overwritten | Unsafe JSON write path | Compare pre and post opencode.json | Merge keys structurally and preserve existing config |

## Validation checklist

1. opencode.json remains valid JSON and preserves unrelated keys.
2. At least one skill appears in available skills metadata.
3. MCP server list shows configured server and expected auth state.
4. One custom tool executes successfully from .opencode/tools/.
5. Plugin registration is idempotent when plugin mode is enabled.
6. Uninstall removes only owned artifacts.

## Official references

- https://opencode.ai/docs/skills/
- https://opencode.ai/docs/mcp-servers/
- https://opencode.ai/docs/custom-tools/
