# Trae and Trae-CN Integration

## Use case

Trae profiles in this manual use AGENTS.md as the primary persistent behavior mechanism, with no equivalent hook coverage assumed in the baseline pattern.

## Required assets

- AGENTS.md section with explicit integration rules.
- Optional skill template for command orchestration.

## Installation pattern

Recommended command shape:

```bash
toolctl trae install
toolctl trae uninstall
toolctl trae-cn install
toolctl trae-cn uninstall
```

Anonymous example command:

```bash
toolctl trae install
toolctl trae-cn install
```

Observed baseline behavior:

- Both trae and trae-cn rely on AGENTS guidance.
- PreToolUse-style hook behavior is not assumed.

## Source-anchored implementation deep dive

Reference implementation anchors:

- Trae and trae-cn command paths and AGENTS install flow in installer control logic.
- Shared Trae skill template and fallback behavior guidance.
- Integration tests validating trae and trae-cn target path behavior.

What to replicate in a new CLI:

1. Assume AGENTS is the primary enforcement channel.
2. Encode pre-answer checks and post-edit refresh behavior explicitly in AGENTS section.
3. Add project status command to validate required context artifacts.
4. Keep fallback behavior explicit when hook equivalents are unavailable.

## Fallback-first design

Without reliable hook surfaces, instruction files must carry stronger behavioral controls:

- explicit pre-answer context checks,
- explicit post-edit refresh command,
- explicit escalation path when context artifacts missing.

## Best practices

- Keep AGENTS sections short and enforceable.
- Put complex implementation detail in docs, not in AGENTS body.
- Include verification command users can run to confirm integration state.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Rules seem ignored | AGENTS file not in active discovery path | Check current working directory and instruction chain | Move AGENTS to active project path |
| Context checks skipped | AGENTS section lacks explicit pre-answer step | Review installed AGENTS section text | Add explicit context-first requirement |
| Inconsistent behavior between machines | Different local overrides in environment | Compare project and local instruction files | Normalize to repo-owned AGENTS baseline |
| Trae-CN mismatch with Trae behavior | One variant not updated during install | Verify both install commands and generated files | Re-run installs for both variants |

## Validation checklist

- Install/uninstall idempotent for both trae and trae-cn flows.
- Docs clearly communicate hook limitation and fallback strategy.
- Integration status command reports AGENTS presence and version markers.
