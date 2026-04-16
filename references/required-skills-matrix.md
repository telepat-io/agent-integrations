# Required Skills Matrix

This matrix defines what a new CLI project should ship as skills or reusable prompt modules for each framework profile.

## Skill tiers

- Core required: should exist for all integrations.
- Framework-required: needed to keep behavior stable on a specific host.
- Optional: valuable but not strictly required for v1.

## Core required skills

| Skill | Purpose | Required |
| --- | --- | --- |
| install-bootstrap | Install-time environment checks and path detection | Yes |
| extraction-orchestration | Chunking, dispatch, merge, retry rules | Yes |
| context-query | Query and summarize indexed context artifacts | Yes |
| safety-policy | Destructive action policy and approval gates | Yes |
| diagnostics | Status and troubleshooting instructions | Yes |

## Framework-required extensions

| Framework | Required extension skill | Why |
| --- | --- | --- |
| Claude Code | hook-aware policy skill | Rich hook lifecycle should be explicitly leveraged |
| Codex | AGENTS-layering and hooks skill | Discovery order and hook caveats need explicit handling |
| Gemini | function-schema and mcp-session skill | Function id mapping and MCP mode differences |
| OpenCode | plugin-injection skill | Plugin registration and pre-tool context behavior |
| OpenClaw | deterministic-orchestration and fallback skill | Skill visibility, command surfaces, and profile-specific runtime caveats need explicit operational guardrails |
| Cursor | concise-rule authoring skill | alwaysApply rules should remain short and deterministic |
| Factory Droid | Task-tool fanout skill | Parallel dispatch semantics depend on task batching |
| Trae and Trae-CN | AGENTS-strong fallback skill | No hook-equivalent assumption in baseline profile |
| VS Code MCP host | mcp-config and trust skill | mcp.json scope, trust prompts, and sandbox behavior |

## Optional skills

| Skill | Value |
| --- | --- |
| docs-freshness | Enforces use of current docs MCP sources where available |
| benchmark-and-cost | Estimates token and latency cost of integration decisions |
| migration-assistant | Helps users upgrade from old instruction templates |
| marketplace-curation | Applies consistent intake rules for third-party skill directories |

## Skill packaging guidance

- Keep skill files modular and versioned.
- Include trigger conditions and expected outputs.
- Include explicit fallback behavior when platform capabilities are missing.
- Include one verification prompt per skill to validate activation.
- Include at least one near-miss negative prompt per skill to validate non-activation boundaries.
- Keep `SKILL.md` concise and push conditional detail to `references/`.
- If scripts are bundled, enforce non-interactive interfaces and deterministic output contracts.
- For directory-sourced skills, require source and security review before install.
- For promoted skills, capture baseline comparison evidence showing skill value over no-skill or previous version.

See [Skills Marketplace Integration](../topics/skills-marketplace-integration.md) for a neutral intake workflow.

## Verification prompts

Use these prompts after install to confirm skills are active:

- "List active integration rules for this project and where they were loaded from."
- "Show the expected pre-tool policy for shell commands in this workspace."
- "Describe how this project should refresh context after code edits."

Near-miss negative prompts (examples):

- "I need a broad architecture summary only, no install or config changes." (should not trigger install-bootstrap style skills)
- "List unrelated framework release notes." (should not trigger extraction-orchestration or diagnostics skills)
