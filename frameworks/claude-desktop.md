# Claude Desktop Integration

## Use case

Claude Desktop is a strong integration target for local MCP workflows because it supports:

- one-click desktop extension installs from a directory,
- custom local extension installs from `.mcpb` bundles,
- extension-level configuration UX for required fields and secrets,
- enterprise admin controls for extension allowlists and custom internal bundles.

## Required assets for a CLI integration

- A packaged `.mcpb` bundle containing your local MCP server and `manifest.json`.
- A valid MCPB manifest with accurate runtime/server metadata.
- User-facing setup docs for required config fields (for example API keys or filesystem paths).
- Troubleshooting guidance for extension install, permissions, and missing tools.

## Installation pattern

Recommended command shape:

```bash
toolctl claude-desktop pack
toolctl claude-desktop validate
toolctl claude-desktop doctor
```

Manual install flow in Claude Desktop:

1. Open Settings > Extensions.
2. Click Advanced settings.
3. In Extension Developer, click Install Extension....
4. Select your `.mcpb` file and complete configuration prompts.

Observed baseline behavior from official docs:

- Directory-based extensions can be installed via Settings > Extensions > Browse extensions.
- Custom `.mcpb` files can be installed directly from local disk.
- Installed extensions become available in conversations after successful install/config.

## Source-anchored integration deep dive

Reference implementation anchors for this profile:

- Claude support docs for desktop extension install and troubleshooting.
- MCPB repository docs for manifest schema and CLI packaging/signing workflows.

What to replicate in a new CLI:

1. Add a deterministic pack flow (`validate` then `pack`) for extension bundles.
2. Generate clear post-pack instructions with exact Claude Desktop install steps.
3. Treat user config fields and secret fields as first-class setup UX, not ad hoc env docs.
4. Add a `doctor` command that checks the most common install failures before support escalation.

## Configuration strategy

For desktop extensions, runtime configuration should be declarative in `manifest.json` via `user_config` and referenced through `mcp_config` variable substitution:

- Use `${user_config.KEY}` for values collected in the extension settings UI.
- Mark sensitive secrets with `"sensitive": true` so hosts can store them securely.
- Keep required fields explicit (`"required": true`) to avoid ambiguous startup failures.
- Prefer env injection for credentials and args for non-sensitive path values.

## MCPB packaging strategy

For local extension delivery, package your server as MCPB:

1. Initialize/author `manifest.json`.
2. Validate manifest against schema.
3. Pack bundle into `.mcpb`.
4. Optionally sign the bundle for distribution trust workflows.
5. Verify signature before release.

Example flow:

```bash
mcpb validate .
mcpb pack . my-extension.mcpb
mcpb sign my-extension.mcpb --self-signed
mcpb verify my-extension.mcpb
```

## Admin and enterprise controls

Team and Enterprise operators can:

- enable/disable public extension availability,
- upload custom `.mcpb` extensions for org users,
- enforce org controls that interact with machine-level enterprise policies.

Operational note:

- Machine-level enterprise policy controls can override in-app extension controls.

## Security notes

- Treat every desktop extension as local code execution with user privileges.
- Review third-party `.mcpb` bundles before install.
- Never hardcode secrets into manifest defaults or server code.
- Use sensitive config fields and host secure storage for API keys.
- Provide privacy policy URLs when extensions send user data to external services.
- For Claude Desktop skill execution, assume an isolated runtime with restricted outbound URL access. In practice, allowed domains are primarily package registries and Anthropic endpoints (for example `api.anthropic.com`, `pypi.org`, `npmjs.org`, and `github.com`), so workflows that require arbitrary web access should not be treated as baseline-capable.

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Extension will not install | Corrupt/invalid `.mcpb` or old app version | Re-download bundle and check Claude Desktop version | Repack bundle and update app |
| Extension appears installed but tools are missing | Missing required extension settings or stale registry state | Open extension settings and review required fields | Complete config and restart Claude Desktop |
| Configuration errors at runtime | Invalid file paths or credentials | Validate configured paths and auth values | Correct settings and retry |
| URL/network requests fail unexpectedly | Runtime domain restrictions in isolated skill environment | Check whether destination host is in allowed domains | Route through supported endpoints or move network-dependent steps outside the skill runtime |
| Permission/security warning on install | OS security policy or enterprise policy conflict | Check OS security settings and org policy state | Adjust policy/permissions and reinstall |
| Server conversion confusion | Existing MCP server not packaged as MCPB | Confirm `manifest.json` exists and package output is `.mcpb` | Use `mcpb pack` workflow |

## Validation checklist

- Pack flow validates manifest before creating `.mcpb`.
- Bundle installs through Settings > Extensions > Install Extension....
- Required `user_config` fields are enforced and documented.
- Sensitive fields are marked and not logged in plaintext diagnostics.
- Tools become visible from Claude Desktop connector/tool UI.
- Troubleshooting docs include restart, config validation, and permission checks.
- Enterprise rollout docs include allowlist and policy override caveats.

## Official references

- https://support.claude.com/en/articles/10949351-getting-started-with-local-mcp-servers-on-claude-desktop
- https://support.claude.com/en/articles/12512198-how-to-create-custom-skills
- https://github.com/modelcontextprotocol/mcpb
- https://github.com/modelcontextprotocol/mcpb/blob/main/MANIFEST.md
- https://github.com/modelcontextprotocol/mcpb/blob/main/CLI.md
