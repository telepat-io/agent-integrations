# Framework Weakness Backlog

This backlog tracks documentation gaps against the setup-parity rubric:

- explicit file paths,
- concrete config examples,
- executable command examples,
- verification steps,
- troubleshooting coverage,
- explicit support status for skills and MCP.

## Priority summary

| Priority | Framework file | Primary gap |
| --- | --- | --- |
| High | `frameworks/codex.md` | Needed concrete file-by-file skills/MCP/hooks/plugins guidance (now upgraded in this pass; re-review pending) |
| High | `frameworks/cursor.md` | Rewritten in this pass; needs final parity re-review |
| High | `frameworks/opencode.md` | Rewritten in this pass; needs final parity re-review |
| High | `frameworks/vscode.md` | Renamed and expanded in this pass; re-review needed for final parity check |
| High | `frameworks/trae.md` | Rewritten in this pass; needs final parity re-review |
| High | `frameworks/factory-droid.md` | Rewritten in this pass; needs final parity re-review |
| High | `frameworks/hermes-agent.md` | Rewritten in this pass; needs final parity re-review |
| Medium | `frameworks/chatgpt-desktop.md` | Needs stronger happy-path verification and install-state checks |
| Medium | `frameworks/claude-code.md` | Needs fuller troubleshooting matrix and setup verification depth |
| Medium | `frameworks/openclaw.md` | Verify support matrix and ensure parity with current official docs |
| Low | `frameworks/claude-desktop.md` | Baseline quality reference |
| Low | `frameworks/gemini.md` | Strong baseline with minor verification hardening |

## Required follow-up edits

Cross-link rule:

- Every backlog item should point to a current workaround section in a framework guide or topic reference.
- If no workaround exists yet, mark status as `Blocked: no documented workaround`.

### 1. Cursor

Target: `frameworks/cursor.md`

Status: Implemented in this pass. Re-review checklist:

- verify project and user scope file paths match latest docs,
- verify hook precedence and matcher caveats are explicitly documented,
- verify verification steps include hooks diagnostics and MCP status checks.

Sources:

- https://cursor.com/docs/skills
- https://cursor.com/docs/hooks
- https://cursor.com/docs/mcp

Workaround references:

- `frameworks/cursor.md`
- `topics/instructions-and-skills.md`

### 2. OpenCode

Target: `frameworks/opencode.md`

Status: Implemented in this pass. Re-review checklist:

- verify tool naming and multi-export examples align with current runtime behavior,
- verify oauth/debug command guidance remains current,
- verify permissions guidance is explicit enough for sensitive environments.

Sources:

- https://opencode.ai/docs/skills/
- https://opencode.ai/docs/mcp-servers/
- https://opencode.ai/docs/custom-tools/

Workaround references:

- `frameworks/opencode.md`
- `references/install-idempotency.md`

### 3. VS Code

Target: `frameworks/vscode.md`

Continue hardening:

- validate final scope parity across skills/hooks/mcp/plugins,
- verify all file paths and preview caveats are current,
- add any missing enterprise/admin constraints if needed.

Sources:

- https://code.visualstudio.com/docs/copilot/customization/agent-skills
- https://code.visualstudio.com/docs/copilot/customization/mcp-servers
- https://code.visualstudio.com/docs/copilot/customization/hooks
- https://code.visualstudio.com/docs/copilot/customization/agent-plugins

Workaround references:

- `frameworks/vscode.md`
- `references/transport-and-auth-matrix.md`

### 4. Trae

Target: `frameworks/trae.md`

Status: Implemented in this pass. Re-review checklist:

- verify skill precedence notes (`.trae/skills` vs `.agents/skills`) remain current,
- verify project MCP toggle and `.trae/mcp.json` behavior against latest IDE UI,
- verify install-link schema examples remain valid.

Sources:

- https://docs.trae.ai/ide/skills
- https://docs.trae.ai/ide/model-context-protocol?_lang=en
- https://docs.trae.ai/ide/add-mcp-servers?_lang=en
- https://docs.trae.ai/ide/mcp-server-install-links?_lang=en
- https://docs.trae.ai/ide/use-mcp-servers-in-agents?_lang=en

Workaround references:

- `frameworks/trae.md`
- `topics/instructions-and-skills.md`

### 5. Factory Droid

Target: `frameworks/factory-droid.md`

Status: Implemented in this pass. Re-review checklist:

- verify hooks storage details in `~/.factory/settings.json` against latest runtime,
- verify MCP layering and `/mcp` behavior are accurately described,
- verify skill compatibility path references remain current (`.agent/skills`).

Sources:

- https://docs.factory.ai/cli/configuration/skills
- https://docs.factory.ai/cli/configuration/mcp
- https://docs.factory.ai/cli/configuration/hooks-guide

Workaround references:

- `frameworks/factory-droid.md`
- `references/install-idempotency.md`

### 6. Hermes Agent

Target: `frameworks/hermes-agent.md`

Status: Implemented in this pass. Re-review checklist:

- verify plugin registration examples remain aligned with current `plugin.yaml` and `register(ctx)` behavior,
- verify skills source-of-truth and external directory precedence notes remain current,
- verify MCP filtering and OAuth config semantics against latest reference docs.

Sources:

- https://hermes-agent.nousresearch.com/docs/user-guide/features/skills
- https://hermes-agent.nousresearch.com/docs/reference/mcp-config-reference
- https://hermes-agent.nousresearch.com/docs/guides/use-mcp-with-hermes
- https://hermes-agent.nousresearch.com/docs/guides/work-with-skills
- https://hermes-agent.nousresearch.com/docs/guides/build-a-hermes-plugin

Workaround references:

- `frameworks/hermes-agent.md`
- `references/transport-and-auth-matrix.md`

## Done in current implementation pass

- Renamed VS Code guide: `frameworks/vscode-mcp-host.md` -> `frameworks/vscode.md`.
- Updated README framework link to the new VS Code guide path.
- Rewrote `frameworks/codex.md` with concrete setup instructions for skills, hooks, mcp, plugins, and plugin build workflows.
- Expanded `frameworks/vscode.md` from MCP-only to multi-surface VS Code customization guidance.
- Rewrote `frameworks/cursor.md` to cover skills, hooks, mcp, and rules with concrete file and config examples.
- Rewrote `frameworks/opencode.md` to cover skills, mcp servers, custom tools, and optional plugin registration.
- Rewrote `frameworks/trae.md` to cover skills, MCP setup, project-level MCP config, and install-link workflows.
- Rewrote `frameworks/factory-droid.md` to cover skills, MCP, and hooks with concrete commands and config examples.
- Rewrote `frameworks/hermes-agent.md` with concrete skills, MCP config, and plugin setup guidance anchored to current official docs.
