# Agent Integration Manual for CLI Projects

This manual documents best practices for integrating a CLI tool with modern coding-agent ecosystems, based on:

- reference implementation patterns.
- a companion state service integration pattern for MCP and lifecycle hooks.
- official platform docs for Claude Code, Codex, Gemini, MCP, VS Code MCP host behavior, and Context7.

## Audience

- Maintainers building a new CLI that should be agent-compatible from day one.
- Platform integrators maintaining hooks, instruction files, and skills across tools.
- Security and infra reviewers validating trust boundaries around tool execution.

## What this manual covers

- Platform-specific setup and installation flows.
- Instruction layering (AGENTS.md, CLAUDE.md, GEMINI.md, Cursor rules, skill files).
- Skills marketplace discovery and skill intake governance.
- Skill authoring lifecycle: quickstart, SKILL.md contract, scripts, and versioning.
- Trigger-quality and output-quality skill evaluation loops.
- Hook lifecycle and guardrails.
- MCP server design and transport choices.
- Security and approvals strategy.
- Observability and integration testing patterns.
- Copy-paste snippets and project bootstrap checklist.

## Manual map

### Framework guides

- [Claude Code](frameworks/claude-code.md)
- [Codex](frameworks/codex.md)
- [Gemini CLI and API](frameworks/gemini.md)
- [OpenCode](frameworks/opencode.md)
- [OpenClaw](frameworks/openclaw.md)
- [Cursor](frameworks/cursor.md)
- [Factory Droid](frameworks/factory-droid.md)
- [Trae and Trae-CN](frameworks/trae.md)
- [VS Code as MCP Host](frameworks/vscode-mcp-host.md)

### Deep technical topics

- [Integration Architecture Patterns](topics/architecture-patterns.md)
- [MCP Server Design](topics/mcp-server-design.md)
- [Hooks and Lifecycle Control](topics/hooks-and-lifecycle.md)
- [Instruction Layers and Skills](topics/instructions-and-skills.md)
- [Skills Marketplace Integration](topics/skills-marketplace-integration.md)
- [Security and Approvals](topics/security-and-approvals.md)
- [Observability and Testing](topics/observability-and-testing.md)

### References

- [Source and Evidence Matrix](references/source-matrix.md)
- [New CLI Integration Checklist](references/new-cli-checklist.md)
- [Required Skills Matrix](references/required-skills-matrix.md)
- [Snippets Cookbook](references/snippets-cookbook.md)
- [Compatibility Matrix](references/compatibility-matrix.md)
- [Docs Maintenance and Changelog](references/docs-maintenance-changelog.md)

### External references used

- [Agent Skills Quickstart](https://agentskills.io/skill-creation/quickstart)
- [Agent Skills Best Practices](https://agentskills.io/skill-creation/best-practices)
- [Optimizing Skill Descriptions](https://agentskills.io/skill-creation/optimizing-descriptions)
- [Evaluating Skills](https://agentskills.io/skill-creation/evaluating-skills)
- [Using Scripts in Skills](https://agentskills.io/skill-creation/using-scripts)
- [Agent Skills Specification](https://agentskills.io/specification)
- [OpenAI Codex Skills](https://developers.openai.com/codex/skills)
- [Anthropic skill-creator SKILL.md](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)
- [OpenClaw Skills](https://docs.openclaw.ai/tools/skills)
- [OpenClaw Creating Skills](https://docs.openclaw.ai/tools/creating-skills)
- [OpenClaw Skills Config](https://docs.openclaw.ai/tools/skills-config)
- [OpenClaw Slash Commands](https://docs.openclaw.ai/tools/slash-commands)
- [OpenClaw ClawHub](https://docs.openclaw.ai/tools/clawhub)

Key skill docs to start with:

- [Instruction Layers and Skills](topics/instructions-and-skills.md) for skill authoring, triggering, scripts, and versioning.
- [Observability and Testing](topics/observability-and-testing.md) for trigger and output eval loops.
- [Skills Marketplace Integration](topics/skills-marketplace-integration.md) for intake, hardening, and promotion gates.

## Implementation model used in this manual

The recommended integration stack for a CLI tool has four layers:

1. Runtime interface layer: CLI commands, config paths, install/uninstall commands.
2. Instruction layer: persistent files that influence agent behavior (AGENTS.md and platform-specific variants).
3. Lifecycle layer: hooks that fire before and after tool execution and at stop/compaction boundaries.
4. Context exchange layer: MCP server for discoverable tools and structured argument schemas.

## Design goals

- Deterministic installation and idempotent reinstall behavior.
- Clear boundaries between policy, execution, and transport.
- Minimal surprise for users switching between agent frameworks.
- Explicit fallback behavior for frameworks with partial hook support.
- Testable integration contracts, not only happy-path docs.

## Current status

This is v1 of the manual and is intentionally implementation-heavy. If you are starting from scratch, use the checklist first, then implement one framework end-to-end before expanding to others.
