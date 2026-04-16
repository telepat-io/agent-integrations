# Factory Droid Integration

## Use case

Factory Droid style integrations often rely on instruction files and task-tool orchestration rather than rich host-side hook APIs.

## Required assets

- AGENTS.md section with deterministic orchestration instructions.
- Skill template optimized for Task-based delegation patterns.

## Installation pattern

Recommended command shape:

```bash
toolctl droid install
toolctl droid uninstall
```

Anonymous example command:

```bash
toolctl droid install
toolctl droid uninstall
```

Observed baseline behavior:

- AGENTS.md is primary always-on mechanism.
- Skill guidance uses Task dispatch for chunked extraction.

## Source-anchored implementation deep dive

Reference implementation anchors:

- Droid command routing and AGENTS section injection in installer control flow.
- Task-based extraction orchestration guidance in droid skill templates.
- Install-level checks for droid skill placement and AGENTS behavior in integration tests.

What to replicate in a new CLI:

1. Keep task chunk contract strict and JSON-only on result channel.
2. Dispatch all chunks in one response when the runtime supports that pattern.
3. Validate and merge per-chunk results defensively.
4. Stop and report when failure ratio exceeds threshold.

## Orchestration template

```text
Dispatch one Task per chunk in the same response to maximize concurrency.
Wait for all tasks.
Merge valid JSON outputs only.
Skip failed chunks and report failure ratio.
```

## Best practices

- Make chunk contract strict and machine-parseable.
- Require confidence tagging in extracted relations.
- Define stop thresholds when too many chunks fail.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Runs are sequential | Tasks dispatched over multiple turns | Inspect transcript for dispatch pattern | Dispatch all chunk tasks in one turn |
| Merge crashes on one chunk | Invalid chunk JSON payload | Validate each chunk before aggregation | Skip invalid chunk and continue with warning |
| Extraction quality drifts | Skill prompt no longer matches schema contract | Compare skill output schema with merger expectations | Version schema and update prompt and merger together |
| AGENTS guidance ignored | Wrong working directory or competing instructions | Inspect active instruction file chain | Move AGENTS near work root and remove conflicting overrides |

## Validation checklist

- AGENTS install/uninstall idempotent.
- Task delegation instructions tested on realistic chunk counts.
- Failure-handling and retry strategy documented.
