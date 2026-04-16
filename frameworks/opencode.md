# OpenCode Integration

## Use case

OpenCode can combine instruction files and plugin hooks for CLI-agent workflows. OpenCode integration is typically achieved via AGENTS.md plus plugin registration.

## Required assets

- AGENTS.md section in project root.
- Plugin file in .opencode/plugins.
- Plugin registration in opencode.json.

## Installation pattern

Recommended command shape:

```bash
toolctl opencode install
toolctl opencode uninstall
```

Anonymous example command:

```bash
toolctl opencode install
toolctl opencode uninstall
```

Observed baseline behavior:

- Writes AGENTS.md guidance section.
- Writes .opencode/plugins/toolctl.js.
- Registers plugin path in opencode.json plugin array.

## Source-anchored implementation deep dive

Reference implementation anchors:

- OpenCode install/uninstall command routing and AGENTS handling in installer control flow.
- Plugin template source and registration merge behavior in OpenCode plugin helpers.
- Plugin write, merge, and cleanup checks in integration tests.

What to replicate in a new CLI:

1. Treat plugin file and registration entry as separate owned artifacts.
2. Parse existing opencode.json safely and preserve unrelated keys.
3. Keep plugin script deterministic and short.
4. Add uninstall cleanup for both file and config registration.

## Plugin pattern

Example plugin behavior:

```js
export const ToolPlugin = async ({ directory }) => {
  return {
    "tool.execute.before": async (input, output) => {
      if (input.tool === "bash") {
        output.args.command = "echo '[context hint]' && " + output.args.command;
      }
    },
  };
};
```

## Configuration snippet

```json
{
  "plugin": [
    ".opencode/plugins/toolctl.js"
  ]
}
```

## Safety notes

- Validate opencode.json before write to avoid silent plugin failure.
- Keep plugin logic small and deterministic.
- Avoid output mutation that can change command meaning unexpectedly.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Plugin not loaded | Missing plugin entry in opencode.json | Inspect plugin array and file existence | Re-register plugin path and restart host |
| Config gets overwritten | Unsafe JSON write path | Compare pre and post opencode.json contents | Merge keys structurally and preserve existing config |
| Repeated injected preface text | Plugin registered multiple times | Count duplicate plugin entries | Deduplicate plugin array entries |
| Install succeeds but no behavior change | Wrong tool name or hook event inside plugin | Add temporary logging in plugin | Align event and tool checks with runtime payload |

## Validation checklist

- Install writes plugin file and registration exactly once.
- Existing opencode.json keys are preserved.
- Uninstall removes registration and owned plugin file only.
