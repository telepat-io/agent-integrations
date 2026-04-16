# Hermes Agent Integration

## Use case

Hermes Agent is a strong target when you need both:

- direct agent extensibility inside Hermes (tools, providers, plugin engines, skills), and
- interoperability with external MCP ecosystems.

For CLI maintainers, this gives a practical model for building a layered integration story:

1. Internal extension points for product-native behavior.
2. MCP boundaries for external tools and host compatibility.

## Required assets

- A clear extension decision tree (tool vs skill vs provider vs plugin).
- Standardized checklists for each Hermes extension type.
- Explicit coverage of both MCP directions:
  - Hermes consuming external MCP servers.
  - Hermes exposing itself as an MCP server.

## Integration pattern

Hermes extension work is not one installer flow. It is a set of explicit contribution tracks:

- Add tool modules and toolset registration.
- Add provider wiring across auth, model catalog, runtime resolution, and CLI menus.
- Add gateway platform adapters across config, runner, CLI, toolsets, and docs.
- Add memory/context plugins via provider plugin contracts.
- Add skills with SKILL metadata and secure setup declarations.

## Extension decision tree

Choose extension type before coding:

- Skill: instructions plus existing tools are enough.
- Tool: deterministic runtime behavior, API integration, streaming, or binary handling.
- Provider: first-class model-provider UX and protocol handling in Hermes runtime.
- Platform adapter: new messaging channel integration through gateway.
- Memory plugin: external persistent memory backend.
- Context engine plugin: replacement context-management strategy.
- MCP config: external tools without native Hermes implementation.

## Source-anchored implementation deep dive

Reference Hermes docs for these tracks:

- Adding tools.
- Adding providers.
- Adding platform adapters.
- Building memory provider plugins.
- Building context engine plugins.
- Creating skills.
- MCP feature usage and server mode.

What to replicate in a new CLI integration manual:

1. Keep extension surfaces separate and explicit.
2. Provide file-level touchpoint checklists, not only conceptual guidance.
3. Document verification steps and parity audits for each extension type.
4. Treat runtime discovery and capability negotiation as first-class behavior.

## Hermes extension tracks

### Adding tools

Use a native Hermes tool when behavior must execute through a deterministic code path.

Required touchpoints:

1. tools module with availability check, handler, schema, and registration.
2. toolset registration.
3. optional setup-wizard env metadata when credentials are needed.

Rules to preserve:

- return JSON strings from handlers,
- return error objects instead of uncaught exceptions,
- keep schemas strict and tool scope minimal.

### Adding providers

Only add built-in providers when you need first-class UX and runtime behavior.

Core layering to keep aligned:

1. credential resolution,
2. model catalogs and aliases,
3. runtime provider resolution,
4. CLI model/setup discoverability,
5. auxiliary-model defaults and model metadata.

If provider protocol is not OpenAI-compatible, isolate provider logic in adapter modules and route by explicit api mode.

### Adding platform adapters

Adapters are end-to-end gateway integrations, not isolated files.

Typical subsystem checklist:

1. platform enum and config mapping,
2. adapter implementation,
3. gateway runner routing and auth controls,
4. cross-platform delivery mappings,
5. CLI setup/status integration,
6. tool and toolset updates,
7. tests and docs.

Use parity audits against an established platform to catch hidden integration gaps.

### Memory provider plugins

Use memory plugins for external persistent memory backends.

Contract highlights:

- implement required lifecycle and config methods,
- keep availability checks local and fast,
- keep sync hooks non-blocking,
- use profile-scoped home paths,
- preserve single active external memory provider behavior.

### Context engine plugins

Use context-engine plugins when built-in compression strategy is not sufficient.

Contract highlights:

- implement required engine interface methods,
- maintain token and threshold counters,
- ensure compressed outputs are valid message sequences,
- activate via explicit config engine selection,
- preserve single active context engine behavior.

### Creating skills

Skills are preferred when capabilities are instruction-driven.

Key practices:

- keep SKILL metadata precise for triggering,
- use platform/tool gating to avoid clutter,
- separate secret env requirements from non-secret config fields,
- use helper scripts for deterministic parsing,
- evaluate trigger quality and output quality against baseline.

### MCP integration and server mode

Document both modes clearly:

1. Hermes as MCP client: consumes external stdio or HTTP servers, supports per-server filtering, capability-aware wrappers, reload behavior, and optional sampling controls.
2. Hermes as MCP server: exposes Hermes messaging tools to other MCP hosts in stdio mode.

Do not mix client and server setup examples in the same quickstart block.

## Verification workflow

1. Run targeted tests for the touched extension track.
2. Run an interactive smoke test for affected CLI flows.
3. Validate edge cases: missing credentials, invalid args, disabled capability paths.
4. Confirm docs include both implementation checklist and troubleshooting guidance.

## Security notes

- Keep extension-specific auth and secret flows isolated.
- Do not overexpose tool surfaces when filtering is available.
- Treat MCP servers and plugin code as privileged execution surfaces.

Shared guidance:

- ../topics/security-and-approvals.md
- ../topics/observability-and-testing.md
- ../topics/instructions-and-skills.md
- ../topics/mcp-server-design.md

## Troubleshooting matrix

| Symptom | Likely cause | Verify | Fix |
| --- | --- | --- | --- |
| Feature appears half-wired | One layer was updated but related layers were not | Run a file checklist audit for that extension type | Complete all required touchpoints before testing |
| Works in tests but not in CLI flow | CLI menu wiring or setup path missing | Test both direct command and interactive flow | Add CLI/provider menu integration and rerun smoke checks |
| Tool list is noisy or risky | No filtering or broad exposure policy | Inspect configured include and exclude filters | Whitelist minimal tool surface and disable unneeded wrappers |
| Plugin loads but state leaks across profiles | Hardcoded home path used | Check plugin storage path usage | Use profile-scoped home from runtime kwargs |

## Validation checklist

- Extension type selection rubric documented for contributors.
- Tools, providers, adapters, plugins, skills, and MCP guidance are covered in this framework page.
- MCP guidance covers both client and server operation modes.
- Cross-links to shared security, testing, instructions, and MCP design topics are present.

## Source alignment

Primary upstream sources:

- https://hermes-agent.nousresearch.com/docs/developer-guide/adding-tools
- https://hermes-agent.nousresearch.com/docs/developer-guide/adding-providers
- https://hermes-agent.nousresearch.com/docs/developer-guide/adding-platform-adapters
- https://hermes-agent.nousresearch.com/docs/developer-guide/memory-provider-plugin
- https://hermes-agent.nousresearch.com/docs/developer-guide/context-engine-plugin
- https://hermes-agent.nousresearch.com/docs/developer-guide/creating-skills
- https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp