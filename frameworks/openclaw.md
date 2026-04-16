# OpenClaw Integration

## Use case

Use this profile when your CLI needs predictable integration with OpenClaw skills, commands, and workspace-level instruction files.

Keep scope focused on integration mechanics:

- install and uninstall behavior,
- skill paths and precedence,
- skill configuration and allowlists,
- operational command controls,
- ClawHub-based discovery and updates.

Avoid broad channel-specific walkthroughs unless they directly affect CLI integration behavior.

## Required assets

- AGENTS.md guidance for project policy and fallback behavior.
- Skill directories and `SKILL.md` files for reusable workflows.
- Optional MCP/tool surfaces when your CLI requires external runtime capabilities.

## Install and uninstall pattern

Recommended host command shape for your CLI:

```bash
toolctl claw install
toolctl claw uninstall
```

Expected install outcomes:

- Writes tool-owned OpenClaw instruction sections idempotently.
- Installs or updates required skill templates in the chosen scope.
- Provides a post-install verification command path.

## First-run skill setup (practical quickstart)

Use the native OpenClaw skill flow first:

```bash
openclaw skills search "calendar"
openclaw skills install <skill-slug>
openclaw skills list
```

Then start a new session so the skill snapshot refreshes:

```bash
/new
```

or restart the gateway:

```bash
openclaw gateway restart
```

## Skill locations and precedence

OpenClaw skill discovery order is precedence-driven. Workspace skills override broader scopes, so verify install path before debugging trigger issues.

Effective precedence (high to low):

1. `<workspace>/skills`
2. `<workspace>/.agents/skills`
3. `~/.agents/skills`
4. `~/.openclaw/skills`
5. Bundled skills
6. `skills.load.extraDirs`

For multi-agent environments, combine location precedence with per-agent allowlists under `agents.defaults.skills` and `agents.list[].skills`.

## Minimal config controls to document

At minimum, your integration docs should cover:

- `skills.load.extraDirs`, `skills.load.watch`, `skills.load.watchDebounceMs`
- `skills.entries.<skillKey>.enabled`, `env`, `apiKey`, `config`
- `skills.allowBundled` behavior (bundled skills only)
- per-agent skill visibility with `agents.defaults.skills` and `agents.list[].skills`

Practical caveat:

- `skills.entries.*.env` and `skills.entries.*.apiKey` apply to host runs. For sandboxed sessions, configure sandbox env explicitly in sandbox settings.

## Slash command controls for integrators

Command surfaces matter for operational workflows, diagnostics, and controlled config changes.

Key toggles to document in your CLI guide:

- `commands.text`
- `commands.native`
- `commands.nativeSkills`
- `commands.config`
- `commands.mcp`
- `commands.plugins`
- `commands.debug`
- `commands.restart`
- `commands.allowFrom` and `commands.useAccessGroups`

Useful operational commands:

```text
/commands
/tools [compact|verbose]
/skill <name> [input]
/config show|get|set|unset
/mcp show|get|set|unset
/plugins list|show|enable|disable
/debug show|set|unset|reset
```

## ClawHub workflow (native vs CLI)

Use native OpenClaw commands for day-to-day install/update in active workspace:

```bash
openclaw skills install <skill-slug>
openclaw skills update --all
openclaw plugins install clawhub:<package>
openclaw plugins update --all
```

Use the `clawhub` CLI when you need registry-authenticated workflows (publish, sync, delete/undelete, token auth):

```bash
clawhub login
clawhub search "postgres backups"
clawhub install <skill-slug>
clawhub sync --all
clawhub skill publish ./my-skill --slug my-skill --name "My Skill" --version 1.0.0 --tags latest
```

## Security notes

- Treat third-party skills as untrusted code until reviewed.
- Review bundled scripts and command usage before enabling skills in production profiles.
- Keep destructive operations behind explicit confirmation paths.
- Prefer sandboxed execution for risky tool flows.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Skill not invoked | Wrong install path or allowlist restriction | Check active skill path and effective agent skill set | Reinstall into workspace scope or update `agents.*.skills` |
| New skill not visible yet | Session snapshot not refreshed | Check whether session predates install | Start a new session (`/new`) or restart gateway |
| Commands unavailable | Command surface disabled | Check `commands.*` toggles and sender authorization | Enable required toggles and review allowlist settings |
| ClawHub update behavior differs from expectation | Installed via different flow (native vs clawhub) | Inspect install source and lock metadata | Standardize on one install flow per environment and re-sync |

## Validation checklist

- Install and uninstall actions are idempotent and only manage tool-owned sections.
- Skill install scope and precedence are documented with exact paths.
- Session refresh requirement after skill/config changes is documented.
- Command-surface toggles needed by your CLI are listed and validated.
- ClawHub usage is split clearly between native OpenClaw flows and `clawhub` CLI workflows.

## Official references

- Skills: https://docs.openclaw.ai/tools/skills
- Creating Skills: https://docs.openclaw.ai/tools/creating-skills
- Skills Config: https://docs.openclaw.ai/tools/skills-config
- Slash Commands: https://docs.openclaw.ai/tools/slash-commands
- ClawHub: https://docs.openclaw.ai/tools/clawhub
