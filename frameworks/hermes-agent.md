# Hermes Agent Integration

## Use case

Hermes is a strong integration target when you want one host to combine:

- reusable skills,
- tightly scoped MCP servers, and
- extensible Python plugins (tools, hooks, slash commands, CLI commands).

Use these native Hermes surfaces first. Treat AGENTS.md as supplemental guidance.

## Capability support status

| Capability | Status | Primary files and commands |
| --- | --- | --- |
| Skills | Supported | `~/.hermes/skills/`, optional external dirs from `~/.hermes/config.yaml`, `/skills`, `hermes skills ...` |
| MCP servers | Supported | `~/.hermes/config.yaml` under `mcp_servers`, `/reload-mcp` |
| Plugins | Supported | `~/.hermes/plugins/<plugin-name>/`, `plugin.yaml`, `__init__.py`, `/plugins` |
| Plugin-bundled skills | Supported | `~/.hermes/plugins/<plugin>/skills/*/SKILL.md`, loaded via `skill_view("plugin:skill")` |

## Required files and directories

```text
~/.hermes/
  config.yaml
  skills/
    devops/
      deploy-k8s/
        SKILL.md
        references/
  plugins/
    calculator/
      plugin.yaml
      __init__.py
      schemas.py
      tools.py
      skills/
        my-workflow/
          SKILL.md
```

Optional shared skills directory configured as external source:

- `~/.agents/skills/`

## Installation pattern

Recommended command shape for a wrapper CLI:

```bash
toolctl hermes install
toolctl hermes uninstall
toolctl hermes status
```

Expected install behavior:

1. Create or patch `~/.hermes/config.yaml` safely.
2. Scaffold at least one local skill under `~/.hermes/skills/`.
3. Optionally scaffold one plugin under `~/.hermes/plugins/`.
4. Optionally append an MCP server entry under `mcp_servers`.
5. Never overwrite unrelated user config keys.

## Skills setup

### Skill source of truth

Primary skills directory:

- `~/.hermes/skills/`

Hermes can also scan external skill directories configured in `~/.hermes/config.yaml`:

```yaml
skills:
  external_dirs:
    - ~/.agents/skills
    - ${SKILLS_REPO}/skills
```

External dirs are read-only for discovery. Local skills in `~/.hermes/skills/` take precedence on name conflicts.

### Minimal SKILL.md example

`~/.hermes/skills/devops/deploy-k8s/SKILL.md`:

```md
---
name: deploy-k8s
description: Safely deploy a service to Kubernetes with rollback checks.
version: 1.0.0
metadata:
  hermes:
    category: devops
    tags: [kubernetes, deployment]
---

# Deploy K8s

## When to Use
Use for Kubernetes deployment and rollback tasks.

## Procedure
1. Validate target namespace and image tag.
2. Apply rollout manifest.
3. Watch rollout status.

## Verification
Run kubectl rollout status and health checks.
```

### Skill operations

Useful commands:

```bash
hermes skills list
hermes skills browse
hermes skills search kubernetes
hermes skills install official/research/arxiv
hermes skills check
hermes skills update
```

In chat:

```text
/skills
/skills search docker
/plan design a migration strategy
```

## MCP setup

Hermes MCP config lives in `~/.hermes/config.yaml` under `mcp_servers`.

### MCP config example

```yaml
mcp_servers:
  github:
    command: "npx"
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN: "***"
    enabled: true
    timeout: 120
    connect_timeout: 60
    tools:
      include: [list_issues, create_issue, search_code]
      resources: false
      prompts: false

  docs:
    url: "https://mcp.docs.example.com"
    auth: oauth
    enabled: true
    tools:
      exclude: [delete_doc]
      resources: true
      prompts: true
```

### Filtering rules to document

- `tools.include` is allowlist.
- `tools.exclude` is denylist.
- If both are set, include wins.
- `tools.resources` and `tools.prompts` control Hermes utility wrappers.

### MCP reload and verification

After config edits:

```text
/reload-mcp
```

Validation prompts:

- Tell me which MCP-backed tools are available right now.
- List available resource and prompt wrappers for configured MCP servers.

## Plugin setup

### Minimal plugin structure

```text
~/.hermes/plugins/calculator/
  plugin.yaml
  __init__.py
  schemas.py
  tools.py
```

### Minimal plugin manifest

`~/.hermes/plugins/calculator/plugin.yaml`:

```yaml
name: calculator
version: 1.0.0
description: Math calculator with conversion helpers
provides_tools:
  - calculate
  - unit_convert
provides_hooks:
  - post_tool_call
```

### Registration pattern

`__init__.py` should register tools and hooks via `register(ctx)`.

Key guidance:

- Tool handlers should return JSON strings.
- Tool handlers should accept `**kwargs` for compatibility.
- Plugin failures should fail open (plugin disabled), not crash Hermes.

### Plugin-bundled skills

Plugins can ship `skills/*/SKILL.md` and register with `ctx.register_skill(...)`.
These are namespaced (`plugin:skill`) and do not replace built-in skill names.

## Verification checklist

1. `hermes skills list` shows expected local and installed skills.
2. Slash command invocation works for at least one skill.
3. MCP config loads and `/reload-mcp` succeeds.
4. Filtered MCP tool surface matches include and exclude policy.
5. `/plugins` shows plugin with expected tool and hook count.
6. Plugin tool call succeeds and returns valid JSON payload.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Skill not discovered | Wrong directory or malformed frontmatter | Check `~/.hermes/skills/**/SKILL.md` and metadata | Fix path and frontmatter, then start new session |
| Wrong skill version loaded | Local and external skill name collision | Compare local and external dirs for same `name` | Rename one skill or remove duplicate |
| MCP server configured but tools missing | Over-filtered include or disabled wrappers | Inspect `tools.include/exclude/resources/prompts` | Relax filters and `/reload-mcp` |
| MCP server never connects | Missing runtime, bad URL, or auth issues | Check command/url and auth settings in config | Fix runtime/auth and reload |
| OAuth HTTP server keeps re-authing | Token refresh failure or invalid auth mode | Confirm `auth: oauth` and token state | Re-authorize and verify endpoint metadata |
| Plugin appears but tools fail | Handler does not return JSON string | Inspect handler return payloads | Return `json.dumps(...)` for success and error paths |
| Plugin hook crashes | Hook callback signature mismatch | Check callback params and `**kwargs` | Add compatible signature and fail-safe handling |

## Official references

- https://hermes-agent.nousresearch.com/docs/user-guide/features/skills
- https://hermes-agent.nousresearch.com/docs/reference/mcp-config-reference
- https://hermes-agent.nousresearch.com/docs/guides/use-mcp-with-hermes
- https://hermes-agent.nousresearch.com/docs/guides/work-with-skills
- https://hermes-agent.nousresearch.com/docs/guides/build-a-hermes-plugin
