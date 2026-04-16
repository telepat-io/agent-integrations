# Cursor Integration

## Use case

Cursor favors persistent rules files over deep hook customization in typical project setups. This makes it a good target for instruction-driven CLI integration.

## Required assets

- Rule file in .cursor/rules, usually with alwaysApply enabled.
- Optional project status command that verifies required context artifacts.

## Installation pattern

Recommended command shape:

```bash
toolctl cursor install
toolctl cursor uninstall
```

Anonymous example command:

```bash
toolctl cursor install
toolctl cursor uninstall
```

Observed baseline behavior:

- Creates .cursor/rules/toolctl.mdc.
- Uses frontmatter with alwaysApply: true.

## Source-anchored implementation deep dive

Reference implementation anchors:

- Cursor install and uninstall behavior in cursor helper functions.
- Rule template with alwaysApply frontmatter in rule-template constants.
- Rule write, idempotency, and removal checks in integration tests.

What to replicate in a new CLI:

1. Keep rule content concise and high-signal.
2. Use alwaysApply only for truly global project behavior.
3. Treat rule file as owned artifact with predictable uninstall.
4. Keep deeper technical detail in external docs pages, not in rule body.

## Rule template

```md
---
description: toolctl context rules
alwaysApply: true
---

- Read generated architecture summary before broad codebase answers.
- Run update command after modifying code.
```

## Best practices

- Keep rule text concise and action-oriented.
- Avoid embedding long examples in alwaysApply rules.
- Link to deeper docs from rule text rather than copying everything into rule file.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Rule not applied | Wrong file path or invalid frontmatter | Check .cursor/rules file and YAML block | Correct path and frontmatter keys |
| Repeated instruction noise | Multiple alwaysApply rules with same content | List all rules in .cursor/rules | Consolidate to one canonical rule |
| Old behavior persists | Stale rule template from prior version | Compare current installed file to template source | Reinstall and stamp template version |
| Rules become too verbose | Excessive implementation detail in rule body | Inspect token length and repeated guidance | Move long guidance into manual topic pages |

## Validation checklist

- Rule file is created once and remains stable on reinstall.
- Uninstall removes only owned rule file.
- Rule content references real commands and artifacts.
