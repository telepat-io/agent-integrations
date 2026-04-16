# Docs Maintenance and Changelog

This page defines how to maintain this manual and tracks notable documentation updates.

## Maintenance workflow

1. Source refresh
- Re-check official docs for Claude, Codex, Gemini, MCP, VS Code host, and Context7.
- Re-check implementation-derived reference modules for installer logic, hooks, instructions, and MCP dispatch.

2. Drift detection
- Compare manual claims against current code behavior.
- Compare snippets against actual command interfaces.
- Compare compatibility matrix against latest platform capabilities.

3. Validation
- Confirm every framework page still has:
  - source-anchored deep dive,
  - troubleshooting matrix,
  - actionable validation checklist.

4. Release
- Update Last updated dates where applicable.
- Add changelog entry with summary and impacted pages.

## Review cadence

- Light review: monthly.
- Full integration audit: quarterly.
- Immediate review: after major framework hook/MCP changes.

## Changelog

## 2026-04-17

Summary:
- Rewrote the OpenClaw framework guide to practical quickstart depth with explicit coverage for skill setup, config controls, slash-command controls, and ClawHub workflows.
- Added official OpenClaw source references across framework, topic, and reference pages.
- Aligned OpenClaw wording in compatibility and required-skills matrices to avoid overstating sequential-only behavior.
- Added runnable OpenClaw and ClawHub command snippets for discovery, install, update, and sync workflows.

Impacted pages:
- docs/agent-integrations/frameworks/openclaw.md
- docs/agent-integrations/topics/instructions-and-skills.md
- docs/agent-integrations/topics/skills-marketplace-integration.md
- docs/agent-integrations/README.md
- docs/agent-integrations/references/compatibility-matrix.md
- docs/agent-integrations/references/required-skills-matrix.md
- docs/agent-integrations/references/new-cli-checklist.md
- docs/agent-integrations/references/source-matrix.md
- docs/agent-integrations/references/snippets-cookbook.md
- docs/agent-integrations/references/docs-maintenance-changelog.md

## 2026-04-16

Summary:
- Added neutral SkillsMP directory guidance as an optional marketplace discovery path.
- Added a dedicated integration topic with intake workflow, security review, and optional API automation notes.
- Updated checklist, required skills matrix, compatibility notes, and source matrix to include marketplace guidance.

Impacted pages:
- docs/agent-integrations/README.md
- docs/agent-integrations/topics/skills-marketplace-integration.md
- docs/agent-integrations/references/new-cli-checklist.md
- docs/agent-integrations/references/required-skills-matrix.md
- docs/agent-integrations/references/compatibility-matrix.md
- docs/agent-integrations/references/source-matrix.md
- docs/agent-integrations/references/docs-maintenance-changelog.md

## 2026-04-10

Summary:
- Initial manual implementation completed.
- Added full framework page set.
- Added deep technical topics and reference guides.
- Added source matrix, checklist, required skills matrix, snippets cookbook.
- Added source-anchored deep dives and troubleshooting matrices to each framework page.
- Added compatibility matrix and maintenance workflow.

Impacted pages:
- docs/agent-integrations/README.md
- docs/agent-integrations/frameworks/*.md
- docs/agent-integrations/topics/*.md
- docs/agent-integrations/references/*.md

## Changelog entry template

Date: YYYY-MM-DD
Summary:
- Item 1
- Item 2

Impacted pages:
- path/to/page-a.md
- path/to/page-b.md
