# Source and Evidence Matrix

This matrix tags each major recommendation with provenance.

- Official Spec: protocol-level requirement or normative guidance.
- Official Product Docs: platform behavior documented by platform maintainers.
- Reference Implementation: behavior validated in anonymized implementation profiles and integration tests.
- Inferred Practice: synthesis pattern derived from multiple sources.

## Core protocol and transport

| Topic | Source Type | Primary Evidence |
| --- | --- | --- |
| MCP participant model (host, client, server) | Official Spec | modelcontextprotocol.io architecture docs |
| MCP JSON-RPC lifecycle and capability negotiation | Official Spec | modelcontextprotocol.io architecture and lifecycle docs |
| Stdio transport framing requirements | Official Spec | modelcontextprotocol.io transports docs |
| Streamable HTTP headers/session rules | Official Spec | modelcontextprotocol.io transports docs |

## Claude Code

| Topic | Source Type | Primary Evidence |
| --- | --- | --- |
| Hook event model and matcher behavior | Official Product Docs | code.claude.com hooks reference |
| MCP tool naming in hooks (mcp__server__tool) | Official Product Docs | code.claude.com hooks reference |
| Install and section injection pattern | Reference Implementation | Installer module and markdown section-merge tests |

## Codex

| Topic | Source Type | Primary Evidence |
| --- | --- | --- |
| Hook locations and event model | Official Product Docs | developers.openai.com/codex/hooks |
| AGENTS.md discovery chain and overrides | Official Product Docs | developers.openai.com/codex/guides/agents-md |
| Hook file write and merge behavior | Reference Implementation | Installer and hook-merge tests |

## Gemini

| Topic | Source Type | Primary Evidence |
| --- | --- | --- |
| Function calling and function id mapping | Official Product Docs | ai.google.dev function-calling docs |
| Built-in MCP support and limitations | Official Product Docs | ai.google.dev function-calling MCP section |
| Gemini coding-agent setup and skills | Official Product Docs | ai.google.dev coding-agents docs |
| GEMINI instruction and hook setup pattern | Reference Implementation | Gemini instruction installer and hook registration logic |
| Precompress/precompact checkpoint flow | Reference Implementation | Hook lifecycle module and setup examples |

## OpenCode, OpenClaw, Droid, Trae, Cursor

| Topic | Source Type | Primary Evidence |
| --- | --- | --- |
| OpenCode plugin-based pre-execution context injection | Reference Implementation | Plugin registration and pre-tool command injection template |
| OpenClaw skill loading, precedence, gating, and lifecycle | Official Product Docs | https://docs.openclaw.ai/tools/skills |
| OpenClaw skill creation flow and SKILL.md basics | Official Product Docs | https://docs.openclaw.ai/tools/creating-skills |
| OpenClaw skills config schema and agent allowlists | Official Product Docs | https://docs.openclaw.ai/tools/skills-config |
| OpenClaw command surface controls and command catalog behavior | Official Product Docs | https://docs.openclaw.ai/tools/slash-commands |
| ClawHub-native install/update/publish/sync workflows | Official Product Docs | https://docs.openclaw.ai/tools/clawhub |
| OpenClaw profile-specific fallback behavior | Reference Implementation | Integration profile templates and tests in this manual's reference implementation |
| Droid task-tool dispatch conventions | Reference Implementation | Task orchestration skill template and chunking contract |
| Trae no-hook fallback to AGENTS.md | Reference Implementation | AGENTS-first installer logic and fallback template |
| Cursor alwaysApply rule strategy | Reference Implementation | Cursor rule writer module with alwaysApply template |

## VS Code MCP host behavior

| Topic | Source Type | Primary Evidence |
| --- | --- | --- |
| mcp.json workspace vs user scope | Official Product Docs | VS Code MCP server docs |
| MCP trust model and sandboxing support | Official Product Docs | VS Code MCP server docs |

## Companion memory integration profile

| Topic | Source Type | Primary Evidence |
| --- | --- | --- |
| Tool catalog and dispatch style | Reference Implementation | Memory MCP dispatcher and tool routing layer |
| Stop and precompact block-and-save lifecycle | Reference Implementation | Hook lifecycle module with checkpoint block-and-save flow |
| Claude and Gemini setup examples | Reference Implementation | Companion setup guides for Claude and Gemini |

## Context7 skills

| Topic | Source Type | Primary Evidence |
| --- | --- | --- |
| Skill install workflow and ecosystem usage | Official Product Docs | context7.com and ai.google.dev coding-agents |
| Skill as portability layer across agent hosts | Inferred Practice | Combined platform behavior and skill usage models |

## Hermes extension and MCP model

| Topic | Source Type | Primary Evidence |
| --- | --- | --- |
| Tool authoring contract and discovery model | Official Product Docs | Hermes developer guide: adding tools |
| Provider layering and api-mode branching model | Official Product Docs | Hermes developer guide: adding providers |
| Platform adapter cross-surface checklist and parity audit pattern | Official Product Docs | Hermes developer guide: adding platform adapters |
| Memory provider plugin lifecycle and single-provider rule | Official Product Docs | Hermes developer guide: memory provider plugins |
| Context engine plugin contract and single-engine activation model | Official Product Docs | Hermes developer guide: context engine plugins |
| Skill metadata contract, secure setup fields, and distribution guidance | Official Product Docs | Hermes developer guide: creating skills |
| MCP client/server dual-mode behavior, filtering, and sampling controls | Official Product Docs | Hermes user guide: MCP feature page |

## Skill creation and quality workflow

| Topic | Source Type | Primary Evidence |
| --- | --- | --- |
| Claude-native skill authoring best practices (conciseness, naming, descriptions, anti-patterns) | Official Product Docs | [Anthropic Agent Skills Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) |
| Claude-native skill structure and progressive disclosure model | Official Product Docs | [Anthropic Agent Skills Overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) |
| API-side skill usage and deployment guidance | Official Product Docs | [Anthropic Skills Guide (API)](https://platform.claude.com/docs/en/build-with-claude/skills-guide) |
| Skill quickstart and first-skill workflow | Official Product Docs | [Agent Skills Quickstart](https://agentskills.io/skill-creation/quickstart) |
| Skill authoring best practices (scope, gotchas, templates, validation loops) | Official Product Docs | [Agent Skills Best Practices](https://agentskills.io/skill-creation/best-practices) |
| Description-trigger optimization methodology | Official Product Docs | [Optimizing Skill Descriptions](https://agentskills.io/skill-creation/optimizing-descriptions) |
| Eval-driven quality loop (test cases, assertions, grading, benchmarks, human review) | Official Product Docs | [Evaluating Skills](https://agentskills.io/skill-creation/evaluating-skills) |
| Script usage guidance for skills (one-off commands vs bundled scripts) | Official Product Docs | [Using Scripts in Skills](https://agentskills.io/skill-creation/using-scripts) |
| Agent Skills frontmatter and directory contract | Official Spec | [Agent Skills Specification](https://agentskills.io/specification) |
| Codex skill invocation, discovery paths, and optional metadata | Official Product Docs | [OpenAI Codex Skills](https://developers.openai.com/codex/skills) |
| Skill-creator iterative workflow pattern | Reference Implementation | [Anthropic skill-creator SKILL.md](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md) |

## Direct reference URLs used in this update

- https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices
- https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview
- https://platform.claude.com/docs/en/build-with-claude/skills-guide
- https://agentskills.io/skill-creation/quickstart
- https://agentskills.io/skill-creation/best-practices
- https://agentskills.io/skill-creation/optimizing-descriptions
- https://agentskills.io/skill-creation/evaluating-skills
- https://agentskills.io/skill-creation/using-scripts
- https://agentskills.io/specification
- https://developers.openai.com/codex/skills
- https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md
- https://docs.openclaw.ai/tools/skills
- https://docs.openclaw.ai/tools/creating-skills
- https://docs.openclaw.ai/tools/skills-config
- https://docs.openclaw.ai/tools/slash-commands
- https://docs.openclaw.ai/tools/clawhub
- https://hermes-agent.nousresearch.com/docs/developer-guide/adding-tools
- https://hermes-agent.nousresearch.com/docs/developer-guide/adding-providers
- https://hermes-agent.nousresearch.com/docs/developer-guide/adding-platform-adapters
- https://hermes-agent.nousresearch.com/docs/developer-guide/memory-provider-plugin
- https://hermes-agent.nousresearch.com/docs/developer-guide/context-engine-plugin
- https://hermes-agent.nousresearch.com/docs/developer-guide/creating-skills
- https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp

## Skills marketplace discovery

| Topic | Source Type | Primary Evidence |
| --- | --- | --- |
| Directory-based skill discovery model and positioning | Official Product Docs | skillsmp.com/docs |
| Marketplace quality and security caveats | Official Product Docs | skillsmp.com/docs/faq |
| Search endpoint and quota-aware API automation | Official Product Docs | skillsmp.com/docs/api |

## Context7 integration profile

| Topic | Source Type | Primary Evidence |
| --- | --- | --- |
| Host detection and per-host setup orchestration | Reference Implementation | Setup agent map and detection logic in CLI setup module |
| Non-destructive JSON and TOML config merge strategy | Reference Implementation | Config writer module with JSON merge and TOML section append/replace behavior |
| MCP dual transport and auth handling | Reference Implementation | MCP server module with stdio/http transport options and auth extraction/validation |
| Rule injection fallback strategy (file vs append-marker) | Reference Implementation | Setup templates and installer logic for host-specific rule writes |
| Skill trigger-as-description pattern | Reference Implementation | Context7 skill template and skills docs guidance |
| Multi-client MCP config examples and OAuth endpoint variant | Official Product Docs | Context7 all-clients docs and client-specific setup guides |
| Hook lifecycle callbacks absent in this profile | Reference Implementation | Repository-wide search over docs/code/plugins/skills/rules with no hook event artifacts |

## Confidence notes

- Official-doc-backed sections should be treated as normative unless explicitly marked experimental by the vendor.
- Reference implementation behavior should be treated as proven implementation patterns, not universal platform guarantees.
- Inferred patterns are provided to reduce integration friction but should be validated in your target runtime.
