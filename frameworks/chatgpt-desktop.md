# ChatGPT Desktop Integration

## Use case

ChatGPT Desktop can be used as a high-level integration surface for:

- reusable Skills workflows,
- workspace skill sharing and admin-controlled rollout,
- developer-mode access to remote MCP apps/tools for read and write actions.

For implementation planning, treat this as a product surface with account/plan and admin-policy gating.

## Required assets for a CLI integration

- A skill package that follows the Agent Skills format and clear trigger descriptions.
- Workspace-ready guidance for install/share permissions and RBAC paths.
- Remote MCP server endpoints for developer mode app creation (where used).
- Safety docs for write-action confirmations and prompt-injection risk.

## Installation and setup pattern

Recommended command shape:

```bash
toolctl chatgpt install
toolctl chatgpt uninstall
toolctl chatgpt doctor
```

Product-side setup anchors:

1. Manage skills from profile menu > Skills.
2. Create/install skills from conversation, editor, upload, or workspace share flow.
3. For MCP apps, enable Developer mode in Settings > Apps > Advanced settings.
4. Create apps from remote MCP servers and manage enabled tools per app.

Observed baseline behavior from official docs:

- Skills are reusable/shareable workflows and can include instructions, examples, and code.
- Skills can be created, uploaded, installed, and shared inside a workspace.
- Skills follow the open Agent Skills standard, but do not currently sync across products automatically.

## Source-anchored integration deep dive

Reference anchors for this profile:

- OpenAI help docs for Skills UX, sharing, and enterprise admin controls.
- OpenAI developer docs for Developer mode MCP app setup and confirmation behavior.

What to replicate in a new CLI:

1. Provide deterministic skill packaging/install docs with explicit scope (personal vs workspace).
2. Document admin toggles needed in Enterprise/Edu before rollout.
3. Separate read-only versus write-action tool guidance in prompts and SOPs.
4. Add a `doctor` flow that validates plan eligibility, settings state, and MCP endpoint auth mode.

## Skills strategy

When targeting ChatGPT product workflows:

- keep skills focused and composable,
- write explicit trigger descriptions and expected outputs,
- include examples that reduce ambiguity for tool selection,
- test install/use/share behavior in the same workspace policy context as production users.

Operational caveat:

- Skills availability and publishing/install controls can depend on workspace admin configuration.

## Developer mode and MCP strategy

Developer mode enables full MCP client access to tools, including write actions.

Key integration notes:

- Official rollout docs currently describe Developer mode eligibility on the web for specific account tiers; verify desktop parity in your target workspace before treating this as generally available.
- Supported MCP protocols include SSE and streaming HTTP.
- Authentication options include OAuth, no authentication, and mixed auth profiles.
- Tools can be toggled per app, and app refresh is needed to pull updated tool metadata.
- Write actions require confirmation by default; tools without `readOnlyHint` are treated as write actions.

Prompting guidance for deterministic behavior:

- explicitly name app and tool,
- disallow alternatives when multiple tools overlap,
- prescribe call sequence and payload shape for critical paths.

## Security notes

- Developer mode is explicitly high-risk and should be treated as a privileged integration surface.
- Assume prompt-injection exposure and validate every write-capable workflow.
- Keep confirmations enabled for destructive/high-impact actions.
- Avoid broad trust of third-party MCP servers; review endpoint ownership and auth paths.
- Use least-privilege OAuth scopes and short-lived credentials where possible.

## Admin and compliance controls

For Enterprise and Edu workspaces:

- admins can control whether skills are enabled,
- admins can control publishing and install permissions,
- compliance logs include metadata around skill create/share/update/install events,
- data residency behavior follows workspace settings.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Skills UI missing or limited | Plan/admin gating or feature disabled | Check workspace permissions and roles | Enable skill-related toggles or adjust role |
| Shared skill not installable | Install permission disabled in workspace | Verify admin settings for skill installing | Update RBAC or have admin install |
| App tools not appearing in developer mode | App draft/tool refresh not completed | Check app settings and refresh tool metadata | Refresh app and reselect in conversation |
| Unexpected confirmation prompts for read-like tools | Tool missing `readOnlyHint` annotation | Inspect tool metadata/annotations | Add `readOnlyHint` where appropriate |
| MCP app auth failures | OAuth/static credential mismatch or endpoint auth misconfig | Review auth mode and token flow | Reconfigure app auth and retest initialize/list calls |

## Validation checklist

- Skill package is spec-compliant and trigger descriptions are explicit.
- Workspace install/share flows validated for target role profiles.
- Developer mode app creation tested against real MCP endpoint transport/auth mode.
- Tool refresh behavior documented for changed schemas/descriptions.
- Write-action confirmations and escalation paths documented.
- Security docs include prompt-injection and malicious-MCP risk notes.

## Official references

- https://help.openai.com/en/articles/20001066-skills-in-chatgpt
- https://developers.openai.com/api/docs/guides/developer-mode
