# AGENTS.md Maintenance and Sync Rules

## Objective

Use AGENTS.md as the operational contract for future agents so skill, MCP, and plugin integrations stay synchronized with the real CLI surface.

## Why this topic exists

Integration drift is common when command contracts evolve faster than agent-facing artifacts. Typical failures:

- skill files still describe old flags,
- MCP tool schemas no longer match current command arguments,
- plugin wrappers expose stale option names,
- docs mention behavior that changed in code.

This topic defines a repeatable way to prevent that drift.

## AGENTS.md as the source of maintenance policy

Treat AGENTS.md as a policy layer for maintainers and coding agents. It should define:

- required validation commands before handoff,
- when skill specs must be updated,
- when MCP schemas/tools must be updated,
- when plugin adapters/manifests must be updated,
- required docs updates for user-visible changes.

Keep policy statements explicit and testable.

## Required synchronization rules

When a CLI command contract changes (flags, argument types, defaults, required/optional status, output shape), update all impacted integration artifacts in the same change.

At minimum, check these surfaces:

1. CLI implementation and parser definitions.
2. Skill frontmatter/body and examples.
3. MCP tool metadata and input schema.
4. Plugin adapter manifests and argument mappings.
5. User-facing docs and reference pages.
6. Integration tests that assert argument behavior.

Hard rule:

- If a CLI skill is exported and supported arguments change, the skill must be updated in the same PR/commit as the CLI change.
- AGENTS.md should require agents to check and update the repository-root `<toolname>-skill/SKILL.md` package after each significant code update.

Apply the same rule to exported MCP tools and plugin commands.

## Suggested AGENTS.md section template

Use or adapt this section in project AGENTS.md files.

```markdown
## Integration Artifact Sync (Mandatory)

- If CLI command arguments, defaults, validation, or output contracts change, update all affected:
  - skills (`SKILL.md`, templates, examples)
  - MCP tools (schemas, docs, examples)
  - plugin/adapter manifests and command mappings
  - user docs and CLI references
- If an exported skill or MCP tool is stale after a CLI change, the change is incomplete.
- After each significant code update, re-check and refresh `<toolname>-skill/SKILL.md` if any contract, workflow, or examples drifted.

## Required Validation Before Handoff

- Run lint/typecheck/build/test commands defined in this repository.
- Run integration checks that cover skills/MCP/plugin contracts.
- Verify docs build if documentation changed.
```

## Change workflow for maintainers

1. Modify CLI contract.
2. Locate impacted skills/MCP/plugin definitions.
3. Update argument docs/examples and schema definitions.
4. Add or adjust tests that validate new/changed argument behavior.
5. Update AGENTS.md if policy or validation workflow changed.
6. Run required validation commands and docs build.

## Practical checks to catch drift

Add at least one of these checks:

- snapshot test for exported command metadata,
- schema parity test between CLI parser and MCP input schema,
- docs assertion test for required flags/examples,
- CI gate that fails when command signature changes without matching integration file updates.

## Minimal review checklist

Reviewers should block merge if any answer is no:

- Were all changed CLI arguments reflected in exported skills?
- Were MCP tool schemas/examples updated?
- Were plugin argument mappings/manifests updated?
- Were user docs and command examples updated?
- Do required checks pass with updated tests?

## Documentation scope guidance

When behavior is user-visible, update both:

- quickstart or usage docs (what changed),
- reference docs (exact contract).

When behavior is maintainer-only, update AGENTS.md and contributor docs so future automation stays aligned.

## Anti-patterns

- Updating only code but not integration artifacts.
- Updating docs examples but not validation/schema logic.
- Deferring skill/MCP/plugin updates to a follow-up issue for a contract change already merged.
- Shipping silent breaking changes to argument names without migration notes.

## Recommended ownership model

Assign a clear owner for each artifact class:

- CLI contract owner,
- skills owner,
- MCP/tooling owner,
- docs owner.

Single-owner changes are fine, but responsibility for cross-artifact sync should be explicit in AGENTS.md.