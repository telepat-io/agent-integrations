# Security and Approvals

## Threat model

Agent-integrated CLI tools can execute commands, read files, and call remote services. Integration docs must treat this as a privileged execution surface.

## Core risks

- Prompt injection through tool outputs or remote MCP responses.
- Over-broad hooks that auto-approve dangerous actions.
- Secret exposure in config files and command strings.
- Unsafe remote MCP trust assumptions.

## Required controls

- Principle of least privilege for active tools.
- Approval gates for sensitive actions.
- Input validation for all tool arguments.
- Audit logs for high-impact operations.

## Approval policy design

- Default: ask for sensitive actions.
- Explicitly list safe actions for auto-approval.
- Separate read-only and mutating tool classes.
- Provide emergency override mode with visible warning.

## Hook safety

- Never trust tool input data blindly.
- Quote shell variables consistently.
- Block path traversal and external write targets unless explicit.
- Keep policy hooks synchronous where enforcement is required.

## MCP trust model

- Prefer official provider-hosted servers where possible.
- Require auth for remote servers.
- Log data sent to third-party MCP servers.
- Document data retention implications outside your own runtime.

## Secure config examples

```json
{
  "servers": {
    "myServer": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@example/mcp-server"],
      "sandboxEnabled": true
    }
  }
}
```

## Checklist for release readiness

- Security review performed on install/uninstall scripts.
- Destructive command handling covered in tests.
- Secrets excluded from docs and templates.
- Remote MCP usage includes explicit trust disclaimer.
- Approval policy matrix documented by framework.
