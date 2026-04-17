# Install Idempotency Guide

Use this guide to make integration install, update, and uninstall safe to re-run.

## Goals

- Re-running install should not duplicate entries.
- Re-running uninstall should not remove user-owned config.
- Update should replace only tool-owned sections.

## Core patterns

### 1. Marker-bounded insertion

Use explicit start/end markers for injected sections.

Example:

```text
<!-- AUTO-GENERATED: mytool start -->
...managed content...
<!-- AUTO-GENERATED: mytool end -->
```

Rules:

- Replace only content between matching markers.
- If markers are missing, fail safely with diagnostics.
- Never rewrite unrelated user sections.

### 2. Fingerprint-based ownership

Attach ownership metadata to generated entries.

Example fields:

- `toolId`
- `integrationVersion`
- `installedAt`

Use fingerprints to detect whether an entry should be updated or ignored.

### 3. Dry-run parity

`--dry-run` should report exactly what would change:

- files touched
- entries added/replaced/removed
- conflicts requiring manual action

### 4. Uninstall safety

Uninstall removes only tool-owned artifacts:

- marker-bounded sections
- fingerprint-matching records
- tool-owned files

Never delete entire shared files unless they are fully tool-owned.

## Verification checklist

1. Install twice: second run makes zero net changes.
2. Uninstall twice: second run is a no-op.
3. Install -> uninstall -> install preserves expected state.
4. Mixed file test: unmanaged user content remains intact.
5. Dry-run output matches actual write behavior.

## Failure signals

- Duplicate hooks or duplicated instruction blocks after rerun.
- Missing markers during update path.
- Uninstall removing unrelated user config.

If any signal appears, stop rollout and fix ownership detection before release.
