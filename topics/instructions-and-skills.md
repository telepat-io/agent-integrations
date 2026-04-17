# Instruction Layers and Skills

## Objective

Create a portable instruction and skill model that works across frameworks with different discovery rules.

## Instruction file roles

- AGENTS.md: cross-framework project guidance baseline.
- CLAUDE.md: Claude-specific instructions and memory patterns.
- GEMINI.md: Gemini-specific behavior and tooling hints.
- Cursor rules files: always-apply compact guidance.

## Layering strategy

1. Global user defaults.
2. Repository-level guidance.
3. Subdirectory-specific overrides for specialized workflows.

Prefer additive layering with explicit precedence rules.

## Content contract for instruction sections

Each section should specify:

- pre-answer context retrieval behavior,
- post-edit refresh command,
- fallback behavior when context artifacts are missing,
- operational boundaries for risky actions.

## Skills strategy

Skills provide reusable behavior bundles and are useful for:

- task orchestration templates,
- domain-specific prompts,
- tool invocation conventions.

For Gemini workflows, skills can be combined with docs MCP for freshness.

## Skill authoring quickstart

Use this flow when creating a new skill template for your CLI integrations.

1. Create a directory and `SKILL.md` file in the host-appropriate skill path.
2. Add minimal frontmatter with `name` and `description`.
3. Write task instructions in imperative style, with explicit inputs and outputs.
4. Add one verification prompt to confirm the skill activates.
5. Add one near-miss prompt to confirm the skill does not over-trigger.

Minimal template:

```yaml
---
name: install-bootstrap
description: Use this skill when users are installing or repairing CLI agent integrations, even if they only mention setup errors or missing config files.
---
```

```markdown
1. Detect the current framework and scope (repo or user).
2. Install required instruction, skill, and MCP config artifacts.
3. Print a verification checklist with exact follow-up commands.
```

## SKILL.md contract and structure

Use the Agent Skills specification as the baseline contract for portable skills.

- Required frontmatter fields:
	- `name`: lowercase letters, numbers, and hyphens only; should match directory name.
	- `description`: concise trigger contract for when the skill should be used.
- Optional fields (use only when needed):
	- `license`
	- `compatibility`
	- `metadata`
	- `allowed-tools` (experimental, host support varies)
- Body content:
	- workflow steps,
	- examples,
	- edge-case handling.

Anthropic-specific hardening guidance for Claude-native skills:

- Prefer gerund-style names that communicate action clearly (for example, `processing-pdfs`, `testing-code`).
- Keep names specific; avoid vague names such as `helper`, `utils`, or `tools`.
- Write `description` in third person and include both what the skill does and when to use it.
- Keep description concise and concrete, with user-intent trigger language instead of implementation detail.
- Avoid reserved words in skill names (`anthropic`, `claude`) and keep frontmatter parser-safe.

Reference: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices

Recommended directory shape:

```text
skill-name/
├── SKILL.md
├── scripts/       # optional executable helpers
├── references/    # optional detailed docs loaded on demand
└── assets/        # optional templates and static resources
```

## OpenClaw-specific SKILL.md notes

For OpenClaw-focused integrations, make these constraints explicit in docs and templates:

- `SKILL.md` requires frontmatter `name` and `description`.
- Keep frontmatter parser-safe and concise; avoid multiline complexity in keys that host parsers may not handle consistently.
- OpenClaw supports additional frontmatter behavior toggles such as `user-invocable`, `disable-model-invocation`, and command-dispatch fields for slash-command behavior.
- When using metadata gating (`metadata.openclaw.requires.*`), document expected bins/env/config requirements and fallback behavior.
- If your skill relies on env injection through `skills.entries.*`, clarify host-vs-sandbox behavior in the skill docs.

Useful references:

- https://docs.openclaw.ai/tools/skills
- https://docs.openclaw.ai/tools/creating-skills
- https://docs.openclaw.ai/tools/skills-config
- https://docs.openclaw.ai/tools/slash-commands

## Progressive disclosure patterns

Design skills so context load stays efficient:

1. Metadata layer (`name`, `description`) handles discovery and triggering.
2. `SKILL.md` body carries compact core workflow guidance.
3. `references/` and `assets/` hold deeper material read only when needed.

Guidance:

- Keep `SKILL.md` focused on the default path.
- Keep `SKILL.md` body compact; for Claude-native skills, target under 500 lines.
- Move long or conditional detail to `references/`.
- Include explicit pointers: "Read `references/api-errors.md` when status is non-200."
- Keep references shallow from `SKILL.md` (one-level deep) and avoid chained references.
- Avoid deep reference chains that force unnecessary context loading.

## Description optimization patterns

Description quality is the primary trigger-control surface in most hosts.

Write descriptions that:

- use imperative phrasing ("Use this skill when...");
- for Anthropic skill frontmatter, use third-person phrasing for reliable discovery;
- capture user intent, not implementation details;
- define scope boundaries (what is in and out);
- include near-synonyms and indirect user phrasing;
- remain concise and specific.

Practical loop:

1. Build a trigger eval set with both should-trigger and should-not-trigger prompts.
2. Use realistic prompts (paths, user context, ambiguous wording).
3. Run each prompt multiple times to reduce nondeterminism effects.
4. Improve description by category, not by keyword overfitting.
5. Keep a fixed validation split and select the best iteration by validation score.

## Common anti-patterns to avoid

- Time-sensitive wording that goes stale quickly (for example, date-gated rules in the default path).
- Windows-style backslash paths in docs (`path\\to\\file`) instead of portable forward slashes (`path/to/file`).
- Deep nested references where critical details are only reachable through multiple hops.
- Overly broad option lists without a clear default path.

## Scripts in skills

Prefer instruction-only skills by default. Add scripts when behavior must be deterministic, reusable, or difficult to express reliably in prose.

Script design rules:

- no interactive prompts;
- complete `--help` output with options and examples;
- structured stdout (JSON/CSV/TSV) for machine parsing;
- diagnostics on stderr;
- clear non-zero exit codes with actionable error text;
- safe defaults and optional `--dry-run` for risky operations.

For one-off tools, prefer pinned command invocations in instructions (for example, pinned `npx` or `uvx` usage) and promote repeated logic into `scripts/`.

## Skill versioning and deprecation

Treat skill templates as versioned integration artifacts.

- Keep a version marker near the skill template bundle.
- Document breaking changes in release notes and migration docs.
- Prefer additive updates when possible.
- For breaking trigger changes, ship transition prompts and rollback guidance.
- Re-run trigger and output evals after every version bump.

## Host-specific notes

- Codex:
	- supports explicit and implicit skill invocation;
	- scans repository and user/admin/system skill locations;
	- supports optional `agents/openai.yaml` metadata for UX and invocation policy.
- Anthropic skill-creator pattern:
	- emphasizes iterative draft -> eval -> human review -> revision loops;
	- recommends realistic prompt sets, objective assertions where possible, and baseline comparison;
	- promotes lean instructions with rationale over rigid over-constraint.
- OpenClaw:
	- skill precedence and per-agent allowlists both affect effective skill visibility;
	- session skill snapshots often require a new session (or watcher refresh) after skill/config updates;
	- slash-command availability depends on `commands.*` toggles and authorization settings.

## Skill packaging recommendations

- Keep skill templates versioned and installable.
- Include explicit trigger and expected outputs.
- Keep schema examples embedded for parser consistency.
- Document parallelism assumptions and fallback behavior.

## Discovery and troubleshooting

Common issues:

- Wrong file picked due to override precedence.
- Empty instruction file silently ignored.
- Restart required before skill/index refresh.

Mitigations:

- Provide status command showing active instruction sources.
- Provide doctor command to validate discovered files.
- Keep byte-size limits in mind where host enforces caps.

## Validation checklist

- Instruction insert/remove behavior tested and idempotent.
- Skill install paths validated per platform.
- Discovery order documented with examples.
- Update path documented for versioned templates.

For repository policy that keeps skills, MCP tools, and plugins synchronized with CLI argument changes, see [AGENTS.md Maintenance and Sync Rules](agents-maintenance-and-sync.md).

## Context7-derived integration learnings

### Skills are declarative trigger contracts

- Skill frontmatter `name` and `description` fields can serve as activation cues for documentation workflows.
- Keep trigger descriptions specific to concrete user intents (library setup, API lookup, version-specific usage).
- Treat skill markdown body as execution guidance for tool sequencing, not as a generic prompt dump.

### Cross-host skill portability works with scoped install targets

- Support project and global install scopes for each host.
- Use host-native skill directories when present, and a shared fallback directory where hosts converge.
- Preserve identical core skill content across hosts whenever possible to reduce drift.

### Rule injection needs two modes

- File mode: write dedicated rule files for hosts that support first-class rule directories.
- Append mode: inject bounded sections into host instruction files using explicit markers for safe update/remove.
- Ensure reruns replace only tool-owned sections and never clobber unrelated user instructions.

### Instructions are not a replacement for hooks

- Rules and skills can strongly influence behavior, but they do not provide runtime pre/post tool callbacks by themselves.
- If hook lifecycle is unavailable, document fallback governance in instruction content and MCP tool policies.
