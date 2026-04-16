# MCP Server Design

## Objective

Design an MCP server that is predictable, secure, and easy for agent hosts to consume.

## Minimum protocol contract

Implement these flows first:

- initialize
- tools/list
- tools/call

Keep protocol and tool behavior deterministic before adding advanced features.

## Transport choices

### Stdio

Use stdio for local CLI workflows.

Requirements:

- JSON-RPC messages only on stdout.
- No non-protocol stdout logs.
- Logs on stderr.
- Newline-delimited message framing.

### Streamable HTTP

Use when remote access is required.

Requirements:

- POST for client messages.
- Accept headers for json and event-stream.
- Session id handling where server uses stateful sessions.
- Origin validation and auth controls.

## Tool surface design

- Start with small tool set (prefer fewer than 20 active tools).
- Use strict schemas and enums where possible.
- Keep names stable and semantic.
- Avoid overloaded multi-purpose tools.

## JSON schema quality

Strong schema shape example:

```json
{
  "name": "query_graph",
  "description": "Query indexed graph context for architecture questions.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "question": { "type": "string" },
      "traverse_mode": {
        "type": "string",
        "enum": ["bfs", "dfs"]
      },
      "token_budget": { "type": "integer" }
    },
    "required": ["question"],
    "additionalProperties": false
  }
}
```

## Error model

- Return structured protocol errors.
- Keep internal traces out of user-facing tool output.
- Include actionable reason strings for recoverable failures.

## Capability negotiation

- Declare capabilities honestly.
- Do not advertise unsupported primitives.
- Version gate behavior by negotiated protocol version.

## Security controls

- Authenticate remote callers.
- Treat all tool args as untrusted input.
- Validate file paths and command arguments.
- Add explicit policy for high-impact actions.

## Reference notes

Two independent reference implementations provide useful patterns:

- One profile exposes query-oriented context tools in MCP mode.
- Another profile exposes read/write memory tools with explicit dispatch mapping and protocol metadata.

## Validation checklist

- tools/list output remains stable between minor versions.
- tools/call validates arguments before execution.
- No stdout pollution in stdio mode.
- Transport-specific headers and session semantics are documented and tested.

## Context7-derived integration learnings

These learnings are directly relevant to CLI-to-agent integration and can be reused in other profiles.

### Dual transport profile is practical

- Offer local stdio for environments where local process launch is expected.
- Offer remote HTTP for hosts that prefer centrally managed endpoints.
- Keep both transports exposing the same tool contract to avoid host-specific behavioral drift.

### Dual auth profile reduces host friction

- Support API-key headers for hosts with static secret injection.
- Support OAuth endpoint variants for hosts that can complete interactive auth flows.
- Document transport-auth compatibility clearly (for example, OAuth with remote HTTP, API key with both modes where supported).

### Tool surface should stay minimal and read-only

- Start with a small read-only tool set for documentation retrieval and library resolution.
- Keep mutation tools out of early releases unless they are necessary for the integration objective.
- Use strict input schemas and enforce clear call sequencing when one tool depends on another.

### Host config formats vary more than transport semantics

- Plan for both JSON and TOML config writers in setup automation.
- Keep config merge logic idempotent and non-destructive.
- Use stable server keys and update-in-place behavior so reruns are safe.

### Important scope caveat

- A profile can have strong MCP support without offering runtime hook callbacks.
- Treat hook lifecycle as a separate capability that must be documented independently.
