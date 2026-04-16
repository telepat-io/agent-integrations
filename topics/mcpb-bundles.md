# MCP Bundles (MCPB)

## Objective

Provide a practical, source-anchored guide for packaging local MCP servers as `.mcpb` bundles so they are easier to install, configure, and distribute across desktop hosts.

## What MCPB is

MCPB is a bundle format for local MCP servers:

- file extension: `.mcpb`,
- archive type: zip-based package,
- required contract: a `manifest.json` describing server and capabilities.

From the maintainers, this is the renamed successor to DXT and is intended to make local MCP server distribution more portable across host applications.

## When to use MCPB

Use MCPB when you need:

- single-file distribution for local MCP servers,
- host-managed setup UX for user config fields,
- repeatable installation for teams without manual JSON edits,
- optional signing/verification before broader distribution.

Prefer raw MCP server registration only when your host does not support MCPB packaging yet.

## Minimal bundle model

A minimal bundle requires `manifest.json`. Typical server bundles include runtime files and dependencies.

Example Node-oriented structure:

```text
my-extension.mcpb
├── manifest.json
├── server/
│   └── index.js
├── node_modules/
└── package.json
```

## Manifest essentials

The manifest schema defines required and optional extension metadata.

Required fields:

- `manifest_version`
- `name`
- `version`
- `description`
- `author` (with author name)
- `server`

Commonly used optional fields:

- `display_name`, `long_description`, `keywords`, `license`
- `repository`, `homepage`, `documentation`, `support`
- `tools`, `prompts`, `tools_generated`, `prompts_generated`
- `compatibility`, `privacy_policies`, `localization`, `icons`
- `user_config`, `_meta`

## Server types and runtime model

MCPB supports multiple server types:

- `node`: Node.js server with bundled dependencies.
- `python`: Python server with bundled dependencies.
- `binary`: self-contained executable.
- `uv` (manifest v0.4+): host-managed Python/runtime dependency flow.

Guidance:

- Use the runtime type that matches your portability and bundle-size constraints.
- Keep `compatibility.platforms` and `compatibility.runtimes` explicit when needed.
- For binary bundles, test per-platform artifacts on clean machines.

## mcp_config and variable substitution

`server.mcp_config` defines how hosts launch the bundled MCP server.

Useful substitutions include:

- `${__dirname}` for installed bundle directory,
- `${HOME}`, `${DESKTOP}`, `${DOCUMENTS}`, `${DOWNLOADS}`,
- `${user_config.KEY}` for values collected from user settings.

Implementation tips:

- Use env vars for secrets and args for non-sensitive paths/options.
- Keep command paths portable and avoid host-specific assumptions.
- For platform-specific needs, use `platform_overrides`.

## user_config design

Use `user_config` to collect extension settings through host UI.

Supported config types include:

- `string`, `number`, `boolean`, `directory`, `file`.

Recommended practices:

- mark required fields explicitly,
- set `sensitive: true` for secret strings,
- provide clear `title` and `description` for operator clarity,
- constrain numeric values with `min` and `max` where appropriate.

## CLI lifecycle for bundle developers

Install CLI:

```bash
npm install -g @anthropic-ai/mcpb
```

Core command set:

- `mcpb init [directory]`
- `mcpb validate <manifest-or-directory>`
- `mcpb pack <directory> [output]`
- `mcpb sign <mcpb-file>`
- `mcpb verify <mcpb-file>`
- `mcpb info <mcpb-file>`
- `mcpb unsign <mcpb-file>`

Practical development sequence:

```bash
mcpb init
mcpb validate .
mcpb pack . my-extension.mcpb
mcpb sign my-extension.mcpb --self-signed
mcpb verify my-extension.mcpb
```

## Signing and verification model

MCPB supports detached signatures appended to bundle files.

Operational implications:

- unsigned files remain valid zip bundles,
- signatures can be added/verified/removed without changing core archive content,
- certificate-chain handling is supported for production signing workflows.

For production releases, prefer CA-issued certificates and explicit verification gates in CI/release pipelines.

## Packaging hygiene

`mcpb pack` excludes common development artifacts and supports extra exclusions via `.mcpbignore`.

Use `.mcpbignore` to avoid shipping:

- tests,
- coverage artifacts,
- logs,
- local env files,
- temporary build outputs.

## Security and governance

- Treat bundles as executable software artifacts.
- Run manual review for third-party bundles before installation.
- Avoid embedding credentials in bundle files or defaults.
- Document privacy policy URLs when external services process user data.
- Keep a release log mapping bundle version, source revision, and signature status.

## Relationship to other integration layers

MCPB complements, not replaces, other integration surfaces:

- MCP server design still defines tool contracts and protocol behavior.
- Instruction/skill layers still define behavior guidance and task workflows.
- MCPB specifically standardizes packaging, install UX, and host execution metadata.

## Related pages

- [MCP Server Design](mcp-server-design.md)
- [Instruction Layers and Skills](instructions-and-skills.md)
- [Security and Approvals](security-and-approvals.md)
- [Compatibility Matrix](../references/compatibility-matrix.md)
- [Source and Evidence Matrix](../references/source-matrix.md)

## Official references

- https://github.com/modelcontextprotocol/mcpb
- https://github.com/modelcontextprotocol/mcpb/blob/main/MANIFEST.md
- https://github.com/modelcontextprotocol/mcpb/blob/main/CLI.md
