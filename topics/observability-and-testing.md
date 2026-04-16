# Observability and Testing

## Objective

Make integration behavior measurable and testable so docs stay aligned with reality.

## Observability requirements

- Status command that reports installation state per framework.
- Diagnostic command for missing files, malformed JSON, and stale templates.
- Structured logs for install/update/uninstall and hook registration.
- Error messages that map directly to remediation steps.

## What to monitor

- Instruction file write outcomes.
- Hook registration and parsing outcomes.
- MCP server startup and tools/list health.
- Approval denials and policy-triggered blocks.

## Test pyramid

### Unit tests

- Path routing by platform.
- Idempotent insertion/removal in markdown instruction files.
- JSON merge behavior for hook config files.
- Plugin registration and deregistration behavior.

### Integration tests

- End-to-end install to status to uninstall for each major framework.
- Hook execution smoke test with representative event payloads.
- MCP server initialize and tool call handshake.

### Docs validation tests

- Validate command snippets against actual CLI help output.
- Validate JSON snippets for syntax.
- Validate file paths used in docs exist in repository templates.

## Skill evaluation loop

Use a repeatable loop to validate both triggering behavior and output quality.

1. Trigger evaluation
- Build realistic prompt sets with:
	- should-trigger queries,
	- should-not-trigger near-misses.
- Run prompts across the target model set (for Anthropic: Haiku, Sonnet, and Opus when supported).
- Run each prompt multiple times to account for model nondeterminism.
- Track trigger rates rather than single-pass outcomes.

2. Output evaluation
- For each test case, run:
	- with-skill,
	- without-skill baseline (or prior skill version baseline for upgrades).
- Store outputs, timing, and grading artifacts per iteration.

3. Assertion grading
- Write objective assertions where possible.
- Record pass/fail with evidence, not opinion.
- Use scripts for mechanical checks (file existence, schema validity, row counts).

4. Aggregate and analyze
- Compute pass-rate, time, and token deltas.
- Flag always-pass and always-fail assertions for cleanup.
- Investigate high-variance cases for ambiguous instructions.

5. Human review
- Review outputs for subjective quality dimensions.
- Capture actionable feedback per test case.
- Feed failures and feedback into the next skill revision.

## Model-coverage gate for Claude-native skills

When promoting Claude-native skills, evaluate behavior on each model tier you plan to support.

- Haiku: verify the skill provides enough explicit guidance for reliable execution.
- Sonnet: verify balanced behavior and efficient instruction following.
- Opus: verify the skill avoids over-constraining and unnecessary verbosity.
- Track per-model pass-rate deltas and investigate large variance before promotion.

## Recommended artifacts

Keep evaluation artifacts versioned by iteration:

- `evals/evals.json`: prompts, expected outputs, optional assertions.
- `iteration-N/.../outputs/`: generated files per run.
- `timing.json`: duration and token usage signals where available.
- `grading.json`: assertion-level evidence and pass/fail results.
- `benchmark.json`: aggregate metrics and deltas.
- `feedback.json`: human review notes.

This layout keeps regression analysis and skill tuning auditable.

## Reference evidence

Installation and section-merge integration tests are strong examples of integration contract tests for installer behavior and markdown section handling.

## Failure taxonomy

- Discovery failures: instruction files not loaded.
- Registration failures: malformed hook config.
- Runtime failures: tool call blocked or parser failure.
- Drift failures: docs no longer match code behavior.

## Skill-specific failure patterns

- Under-triggering: relevant prompts do not load the skill.
- Over-triggering: skill loads for adjacent tasks where it should not.
- Instruction bloat: long `SKILL.md` causes weaker task focus.
- Script hang: bundled commands wait for interactive input.
- Baseline regression: skill output is not better than no-skill or previous version.
- Portability drift: skill works on one host but fails on another due to path or policy differences.

## Drift prevention workflow

- Tag docs with source-of-truth links.
- Run docs snippet checks in CI.
- Update matrix when vendor behavior changes.
- Keep compatibility notes per framework version.

## Validation checklist

- At least one integration test per framework family.
- docs build or lint checks include snippet validation.
- Status command output is stable and documented.
- Known limitations are tested and documented, not hidden.
- Trigger eval sets include near-miss negatives, not only easy negatives.
- Output eval compares with an explicit baseline and tracks deltas.
- Human review feedback is captured and tied to a revision decision.
- Claude-native skills are tested across intended Anthropic model tiers before default rollout.

## Platform callouts

- Codex:
	- test both explicit invocation and implicit trigger behavior;
	- validate expected behavior for repo/user/admin/system skill locations in your support matrix.
- Anthropic skill-creator style workflows:
	- use iterative draft -> run -> grade -> review loops;
	- keep assertions objective and pair them with qualitative review before final changes.
