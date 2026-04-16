# New CLI Integration Checklist

Use this checklist to bootstrap agent integrations in a new CLI project.

## Stage 1: Core CLI and packaging

- Define stable CLI entry point and subcommands.
- Add explicit install and uninstall commands per framework.
- Ensure idempotent writes to config and instruction files.
- Add dry-run mode for install and uninstall actions.

## Stage 2: Instruction files

- Create framework instruction templates:
  - CLAUDE.md section template.
  - AGENTS.md section template.
  - GEMINI.md section template.
  - Cursor rule template (.cursor/rules/*.mdc).
- Include project-specific context hints, rebuild instructions, and fallback behavior.
- Add version marker file to detect stale installed templates.

## Stage 3: Hook setup

- Implement hook installers for frameworks that support them.
- Detect malformed JSON and fail safely.
- Make hook registration idempotent.
- Add uninstall path that removes only tool-owned hook entries.
- Document unsupported-hook fallback strategy per platform.

## Stage 4: MCP server

- Expose minimum useful tool set first.
- Define strict JSON schema for all tool args.
- Implement initialize, tools/list, and tools/call correctly.
- Return deterministic error payloads.
- Keep logs on stderr only for stdio transport.
- If shipping local desktop extensions, add MCPB packaging gates (`mcpb validate` and `mcpb pack`) before release.
- If shipping signed bundles, verify signature status in CI (`mcpb verify`) and publish expected trust model.

## Stage 5: Skills and reusable behavior

- Decide whether to ship local skills, remote skills, or both.
- Add install command for skill templates where platform supports it.
- Define marketplace intake policy (allowed sources, review criteria, and rejection rules).
- Provide verification command users can run after install.
- Include troubleshooting for skill discovery and restart requirements.
- Validate every shipped skill against SKILL.md frontmatter and naming contract.
- Add trigger eval set with should-trigger and near-miss should-not-trigger prompts.
- Require non-interactive script behavior (`--help`, explicit flags, deterministic exit codes).
- Require with-skill versus baseline output comparison before default promotion.
- Record source revision, eval summary, and rollback steps for each approved skill.
- For Claude-native skills, align with Anthropic best practices: specific description trigger contract, concise `SKILL.md` core path, and anti-pattern checks.
- For Claude-native skills, validate behavior on each intended Anthropic model tier before default promotion.
- For OpenClaw integrations, verify skill scope and precedence (`<workspace>/skills`, `.agents/skills`, user/global paths) before debugging trigger failures.
- For OpenClaw integrations, document session refresh behavior after skill/config changes (new session or gateway restart when needed).

## Stage 6: Security and approvals

- Default to explicit approvals for sensitive actions.
- Add allowlist and denylist patterns for shell tool calls.
- Never hardcode secrets in docs or config snippets.
- Document trust requirements for remote MCP servers.

## Stage 7: Observability

- Provide command to print integration status for each framework.
- Add diagnostics for missing hooks, missing instruction files, and malformed config.
- Add clear logs for install, update, and uninstall operations.

## Stage 8: Test strategy

- Unit test installer path logic and idempotency.
- Unit test merge behavior for existing instruction files.
- Unit test hook registration and removal.
- Integration test at least one full framework flow end-to-end.
- Validate snippets in docs against actual command behavior.

## Stage 9: Release and maintenance

- Publish compatibility matrix by framework and version.
- Add changelog section for integration-specific changes.
- Add migration notes when template formats change.
- Schedule regular review for MCP and hook spec drift.

## Stage 10: Surface-proof verification gates

- Classify each integration surface as Implemented, Readiness signal, Roadmap-only, or Absent.
- Require artifact proof for every Implemented claim (code path, config template, or installer behavior).
- Verify setup command reruns are idempotent and non-destructive for existing host config files.
- Verify config writers support host-required formats (for example JSON and TOML where needed).
- Verify transport and auth combinations per host (stdio vs remote HTTP, API key vs OAuth endpoints).
- Verify rule installation mode per host (file-based vs append-marker) and validate safe replacement behavior.
- Verify skill install paths for project and global scope.
- Verify description trigger quality with repeat-run rates, not one-off runs.
- Verify bundled scripts separate structured stdout from diagnostics on stderr.
- If a profile has no hook lifecycle, document fallback governance explicitly instead of implying hook behavior.
