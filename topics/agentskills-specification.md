# Agent Skills Specification

## Purpose

Define a single, implementation-focused standard for authoring, validating, and maintaining `SKILL.md` artifacts across agent hosts.

This page consolidates the practical guidance from:

- Agent Skills quickstart,
- best practices,
- description optimization,
- skill evaluation,
- script usage,
- formal specification requirements.

Use this as the canonical reference when writing or reviewing skills in this repository.

## Scope and non-goals

In scope:

- `SKILL.md` structure and trigger-contract quality.
- Skill workflow design and failure handling.
- Script usage boundaries and deterministic execution.
- Repeatable evaluation and promotion checks.

Out of scope:

- Host-specific installation walkthroughs.
- Marketplace or packaging procedures beyond spec compliance.
- Framework-specific UI flows.

For host-specific setup, see the framework guides under `frameworks/`.

## Specification baseline

Treat the Agent Skills specification as the minimum portability contract.

- Required frontmatter:
  - `name`: lowercase letters, numbers, and hyphens only; align with directory name.
  - `description`: concise trigger contract describing when to invoke the skill.
- Optional frontmatter (use only when needed):
  - `license`
  - `compatibility`
  - `metadata`
  - `allowed-tools` (host support varies)
- Body expectations:
  - Default workflow steps.
  - Input/output expectations.
  - Edge-case and fallback handling.

Authoring baseline template:

```yaml
---
name: extraction-orchestration
description: Use this skill when users need deterministic chunking, dispatch, retry, and merge behavior for CLI extraction workflows.
---
```

```markdown
1. Identify source inputs and output contract.
2. Select the default extraction path and retry policy.
3. Emit structured status and final output summary.
4. If a stage fails, follow the fallback path and report actionable next steps.
```

References:

- https://agentskills.io/specification
- https://agentskills.io/skill-creation/quickstart

## Quickstart workflow

Use this sequence for new skills:

1. Define one clear user-intent surface (what problem the skill solves).
2. Create skill directory with `SKILL.md` and optional `scripts/`, `references/`, `assets/`.
3. Write minimal frontmatter with precise trigger scope.
4. Add compact default workflow in the body.
5. Add one verification prompt and one near-miss prompt.
6. Run evaluation loop before promotion.

Recommended directory shape:

```text
skill-name/
├── SKILL.md
├── scripts/       # optional deterministic helpers
├── references/    # optional deep docs loaded on demand
└── assets/        # optional templates/resources
```

Reference:

- https://agentskills.io/skill-creation/quickstart

## Best practices

Authoring guidance:

- Keep skills narrow and composable; avoid multi-domain catch-all skills.
- Keep workflow steps imperative and deterministic.
- Define explicit in-scope and out-of-scope behavior.
- Keep default path short; push conditional depth to `references/`.
- Include fallback behavior for missing tools, data, or permissions.
- Avoid policy ambiguity for risky operations.

Operational guidance:

- Version skill bundles and record behavior deltas.
- Preserve backward-compatible triggers when possible.
- Re-run evaluations after every meaningful edit.

Reference:

- https://agentskills.io/skill-creation/best-practices

## Optimizing descriptions

`description` is the primary trigger-control surface. Optimize for intent matching, not keyword stuffing.

Write descriptions that:

- begin with clear trigger language (for example, "Use this skill when...").
- capture user intent and context, not implementation detail.
- include boundaries (what should not trigger).
- include common near-synonyms used by users.
- remain concise and specific.

Anti-patterns:

- Vague descriptions (for example, "general helper").
- Overly broad language that causes false positives.
- Internal implementation terms that users rarely type.

Reference:

- https://agentskills.io/skill-creation/optimizing-descriptions

## Evaluating skills

Use repeatable evaluation loops before default promotion.

Minimum eval set:

- Should-trigger prompts (positive set).
- Near-miss should-not-trigger prompts (negative set).
- Realistic prompts with ambiguity and context noise.

Evaluation loop:

1. Run baseline (without skill or previous version).
2. Run candidate skill against the same prompt set.
3. Compare trigger precision and output quality.
4. Log regressions and revise description/workflow.
5. Re-run on a fixed validation subset.

Promotion gate suggestions:

- No critical false-positive triggers on negative set.
- Candidate must outperform baseline or previous revision.
- Fallback behavior validated for expected failures.

Reference:

- https://agentskills.io/skill-creation/evaluating-skills

## Using scripts in skills

Prefer prose-only skills by default. Add scripts only when deterministic execution is hard to guarantee from instructions alone.

Script requirements:

- Non-interactive execution.
- `--help` with options and examples.
- Structured stdout for machine parsing.
- Diagnostics on stderr.
- Non-zero exit codes with actionable messages.
- Safe defaults and `--dry-run` for risky operations when relevant.

When scripts are added:

- Keep `SKILL.md` as orchestration contract.
- Keep script interfaces stable and documented.
- Include script failure/fallback behavior in the skill body.

Reference:

- https://agentskills.io/skill-creation/using-scripts

## Portability checklist

Use this checklist before shipping or promoting a skill:

1. Frontmatter validates against required contract (`name`, `description`).
2. Description trigger scope is explicit and bounded.
3. Default workflow has deterministic steps and clear outputs.
4. Negative prompts confirm non-activation boundaries.
5. Fallback behavior is documented and tested.
6. Optional scripts are non-interactive and parseable.
7. Evaluation evidence is recorded with source revision.

## Common failure modes

| Failure mode | Typical cause | Mitigation |
| --- | --- | --- |
| Over-triggering | Broad description language | Tighten intent scope and add near-miss prompts |
| Under-triggering | Description too narrow or jargon-heavy | Add user-language variants and concise scope examples |
| Inconsistent output | Workflow steps are ambiguous | Make output contract explicit and deterministic |
| Fragile automation | Script assumes interactive context | Enforce non-interactive flags and structured output |
| Regression after edits | No baseline comparison | Run candidate vs baseline on fixed eval sets |

## Related repository docs

- [Instruction Layers and Skills](instructions-and-skills.md)
- [Required Skills Matrix](../references/required-skills-matrix.md)
- [New CLI Integration Checklist](../references/new-cli-checklist.md)
- [Skills Marketplace Integration](skills-marketplace-integration.md)

## External references

- https://agentskills.io/specification
- https://agentskills.io/skill-creation/quickstart
- https://agentskills.io/skill-creation/best-practices
- https://agentskills.io/skill-creation/optimizing-descriptions
- https://agentskills.io/skill-creation/evaluating-skills
- https://agentskills.io/skill-creation/using-scripts
