# Hooks and Lifecycle Control

## Objective

Use hooks to enforce guardrails and inject context at predictable lifecycle points without creating brittle automation.

## High-value events

- SessionStart: preload environment or context hints.
- PreToolUse: enforce command policy before execution.
- PostToolUse: add feedback after tool execution.
- Stop and PreCompact: force save/checkpoint flows.

## Design principles

- Keep hooks deterministic and low-latency.
- Prefer narrow matchers over wildcard matchers.
- Use machine-parseable JSON outputs.
- Separate block/deny policy hooks from observability hooks.

## Policy hook example

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./hooks/block_destructive.sh"
          }
        ]
      }
    ]
  }
}
```

Script contract:

- Read input JSON from stdin.
- Emit one JSON object for decision controls.
- Use exit codes according to host behavior.

## Context-injection pattern

Use pre-tool hooks to provide fast reminder context before high-noise operations (search, shell exploration) when generated artifacts already exist.

## Compaction checkpoint pattern

Before compaction or stop boundaries:

- trigger save instructions,
- block until state persisted,
- allow continuation after successful checkpoint.

This pattern is strongly demonstrated by stop and precompact checkpoint hooks in production-style memory integrations.

## Common failures

- Invalid JSON due to shell profile output noise.
- Infinite stop loop due to missing stop-cycle flag.
- Race conditions from concurrent hook handlers.
- Policy bypass from incomplete tool interception coverage.

## Mitigations

- Keep hook scripts isolated from login shell side effects.
- Add explicit loop-break state markers.
- Limit async hooks to non-policy operations.
- Document event coverage limitations by platform.

## Validation checklist

- Policy hooks tested on allowed and denied inputs.
- Stop and precompact flows tested for loop safety.
- Hook outputs validated against host parser expectations.
- Docs include event support matrix per framework.
