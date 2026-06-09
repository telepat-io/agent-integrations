# Pi Integration

## Use case

[Pi](https://pi.dev) is a coding-agent runtime with first-class skills and MCP support. Ideon integrates via:

- skill paths in Pi agent settings,
- stdio MCP registration in Pi `mcp.json`, and
- the community **pi-mcp-adapter** bridge for proxy-mode MCP in Pi sessions.

Treat Pi as a skills + MCP host. Lifecycle hooks are out of scope for the Ideon reference installer.

## Capability support status

| Capability | Status | Primary files |
| --- | --- | --- |
| Skills | Supported | `~/.pi/agent/settings.json` (global), `.pi/settings.json` (project) — `skills[]` array |
| MCP servers | Supported | `~/.pi/agent/mcp.json` (global), `.pi/mcp.json` (project) — `mcpServers` object |
| MCP bridge | Supported | `pi-mcp-adapter` via `pi install npm:pi-mcp-adapter` |
| Hooks | Not used | Pi hook surfaces vary; Ideon defers hooks to a future epic |

## Required files and directories

Global layout:

```text
~/.pi/agent/
  settings.json    # skills[] paths
  mcp.json         # mcpServers.ideon
```

Project layout:

```text
repo-root/
  .pi/
    settings.json
    mcp.json
```

Skill packages are referenced by **absolute path** in `settings.json` (Ideon resolves paths from the installed npm package).

## Installation pattern

Recommended command shape (Ideon reference implementation):

```bash
ideon agent install pi
ideon agent install pi --mcp-skill
ideon agent install pi --project
ideon agent status --json
```

Observed behavior:

- **Default (`--cli-skill`):** appends `skill/ideon-cli` package path to `settings.skills`.
- **`--mcp-skill`:** runs `pi install npm:pi-mcp-adapter`, merges `mcpServers.ideon`, appends `ideon-mcp` skill path.
- Conflicting Ideon-managed entries skip with warning; `--force` replaces Ideon-owned values only.
- Uninstall removes managed skill paths and MCP key; **does not** uninstall pi-mcp-adapter.

## Skills setup

### Settings file

`~/.pi/agent/settings.json`:

```json
{
  "skills": [
    "/path/to/node_modules/@telepat/ideon/skill/ideon-cli"
  ]
}
```

Project-scoped equivalent: `.pi/settings.json` in the repository root when using `ideon agent install pi --project`.

### Skill packages

Ideon ships two installable packages in npm:

- `skill/ideon-cli/` — terminal/CLI workflows
- `skill/ideon-mcp/` — MCP tool catalog and host setup guidance

Prefer symlink from package root to a stable path; fall back to copy when symlinks are unavailable.

## MCP setup

### MCP config

`~/.pi/agent/mcp.json`:

```json
{
  "mcpServers": {
    "ideon": {
      "command": "ideon",
      "args": ["mcp", "serve"],
      "lifecycle": "lazy"
    }
  }
}
```

### pi-mcp-adapter

Install the adapter before relying on MCP tools in Pi:

```bash
pi install npm:pi-mcp-adapter
```

Use **proxy mode** through the adapter. Avoid registering the full Ideon MCP tool surface with `directTools: true` unless the user accepts the large context footprint (~39 tools).

## Trust model

- Pi may require explicit trust for project-scoped settings and MCP servers.
- Ideon does not automate project trust prompts in the reference installer.
- Document manual trust steps when `ideon agent status --json` reports missing artifacts.

## Best practices

- Run `ideon agent status --json` after install to verify `piBinaryOnPath`, skill paths, and MCP config.
- Keep skill paths absolute and stable across npm global upgrades (re-run install after major upgrades if paths change).
- Use `--dry-run` in CI to validate planned mutations without writing host files.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| `pi-missing` in status JSON | `pi` not on PATH | `pi --version` | Install Pi CLI (`npm install -g --ignore-scripts @earendil-works/pi-coding-agent`) |
| Skill not loaded | Wrong path in `settings.skills` | Inspect settings file | Re-run `ideon agent install pi [--force]` |
| MCP tools unavailable | Adapter missing or bad `mcp.json` | Check `pi-mcp-adapter` and `mcpServers.ideon` | `ideon agent install pi --mcp-skill --force` |
| Install skipped with warning | Conflicting Ideon entry | Read install stderr | Re-run with `--force` if replacement is intended |
| pi install adapter timeout | Network or pi subprocess hang | Run `pi install npm:pi-mcp-adapter` manually | Fix network/pi install; retry MCP install |

## Validation checklist

1. `pi --version` succeeds on PATH.
2. `settings.skills` contains Ideon skill path(s) for chosen mode.
3. For MCP mode: `mcp.json` contains `mcpServers.ideon` and pi-mcp-adapter is installed.
4. `ideon agent status --json` reports `readiness.cliSkillLinked` / `mcpConfigured` as expected.
5. Uninstall removes Ideon-managed paths without deleting unrelated Pi configuration.

## Official references

- https://pi.dev/docs/latest
