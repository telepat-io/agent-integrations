# Integration Architecture Patterns

This page describes reusable architecture patterns for agent-compatible CLI tools.

## Pattern 1: Layered integration surface

A robust CLI integration architecture should separate concerns into layers:

1. CLI control plane: install, uninstall, status, and runtime commands.
2. Instruction plane: persistent behavior hints loaded by agent hosts.
3. Hook plane: lifecycle interception for policy enforcement and auto-context injection.
4. Context plane: MCP tool exposure and schema-defined capabilities.

Why it matters:

- Keeps framework-specific glue decoupled from product logic.
- Makes partial support (for example no hooks) easier to handle by fallback.
- Enables testing each layer independently.

## Pattern 2: Capability map per framework

Model each framework by capabilities instead of brand names.

Example capability dimensions:

- Supports instruction files.
- Supports project-local hooks.
- Supports user-global hooks.
- Supports MCP tools directly.
- Supports skills/manifest packages.
- Supports subagent or parallel task dispatch.

Then build install logic from capabilities, not ad-hoc branches.

## Pattern 3: Deterministic install state

Track install state explicitly.

Recommended artifacts:

- Version stamp file for installed templates.
- Marker section headers in instruction files.
- Hook entry fingerprints for safe update and safe removal.

This avoids ambiguous uninstall and prevents deleting user-authored content.

## Pattern 4: Fallback-first behavior

Many frameworks do not support the same hook surface. Design graceful fallbacks.

Examples:

- If hooks unavailable, strengthen instruction templates and startup guidance.
- If MCP unavailable, provide equivalent local command wrappers and query files.
- If subagent parallelism unavailable, make sequential strategy explicit.

## Pattern 5: Thin platform adapters

Keep platform adapters small and declarative.

A practical model:

```python
PLATFORM_CONFIG = {
    "claude": {
        "instruction_target": "CLAUDE.md",
        "hook_target": ".claude/settings.json",
        "skill_template": "skill.md",
    },
    "codex": {
        "instruction_target": "AGENTS.md",
        "hook_target": ".codex/hooks.json",
        "skill_template": "skill-codex.md",
    },
}
```

Then use shared functions for:

- section insert/remove,
- json merge,
- plugin registration,
- status checks.

## Pattern 6: Evidence-driven docs and tests

Treat docs as part of the integration contract.

- Each documented command should have at least one test-backed behavior.
- Each config snippet should map to real implementation logic.
- Docs should separate normative behavior from implementation-derived behavior.

## Anti-patterns

- Hiding framework limitations in marketing language.
- Overloading one instruction file for every framework.
- Mixing transport concerns (stdio/http) with tool semantics.
- Writing hooks that depend on fragile working-directory assumptions.
- Ignoring uninstall and rollback behavior.

## Minimal architecture for v1

If you need to launch fast, implement this minimum set:

1. Install and uninstall command for one framework.
2. One instruction template with marker section and version stamp.
3. One hook registration path with idempotent merge.
4. One MCP server exposing 3-5 high-value tools.
5. Status command showing integration health.
6. Unit tests for install and uninstall behavior.
