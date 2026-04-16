# Compatibility Matrix

Last updated: 2026-04-17

This matrix tracks integration assumptions used in this manual.

## Capability legend

- Full: first-class support documented and used in implementation-derived implementation.
- Partial: supported with caveats or evolving behavior.
- Fallback: not available in baseline profile, handled through instructions/workarounds.

## Reference profile status legend

- Implemented: concrete integration artifacts exist in code or shipped config templates.
- Readiness signal: adjacent implementation exists and lowers future integration effort, but not a full surface.
- Roadmap-only: docs mention support, but no implementation artifacts are present.
- Absent: no explicit support artifacts are present.

## Framework capability matrix

| Framework | Instructions | Hooks | MCP usage | Skills | Parallel task pattern | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| Claude Code | Full | Full | Full | Full | Full | Richest lifecycle hooks and strong policy surface |
| Claude Desktop | Full | Partial | Full | Full | N/A | Local desktop host with extension packaging (`.mcpb`) and admin policy controls |
| Codex | Full | Partial | Full | Full | Full | Hook surface evolving; enforce documented subset |
| Gemini CLI/API | Full | Full | Partial | Full | Full | SDK MCP support marked experimental in docs |
| VS Code MCP Host | N/A host-level | N/A host-level | Full | N/A host-level | N/A host-level | Host runtime for MCP servers; use mcp.json scopes |
| OpenCode | Full | Partial via plugin | Partial | Full | Full | Plugin hook model drives pre-tool injection |
| OpenClaw | Full | Partial profile-dependent | Partial | Full | Partial | Skills and slash-command surfaces are first-class; profile behavior can still require deterministic fallback patterns |
| Cursor | Full | Fallback | Partial | Partial | N/A | Rules-driven model with alwaysApply pattern |
| Factory Droid | Full | Fallback | Partial | Full | Full | Task orchestration and AGENTS-driven controls |
| Trae / Trae-CN | Full | Fallback | Partial | Full | Full | AGENTS-first fallback where hooks unavailable |

## Reference command compatibility snapshot

| Command family | Sample CLI | Sample Memory Service |
| --- | --- | --- |
| claude install | Yes | Via docs/hooks setup |
| codex install | Yes | Via hooks docs pattern |
| gemini install | Yes | Via examples and MCP setup |
| cursor install | Yes | N/A |
| opencode install | Yes | N/A |
| claw install | Yes | N/A |
| droid install | Yes | N/A |
| trae install | Yes | N/A |
| mcp server run | Yes | Yes |

## Context7 reference profile (integration surfaces)

| Surface | Status | Why it matters for CLI integrators | Evidence class |
| --- | --- | --- | --- |
| MCP transport | Implemented | Demonstrates dual local and remote onboarding paths for hosts with different trust/network models | Reference Implementation |
| MCP auth model | Implemented | Shows practical OAuth vs API-key selection and host-specific header wiring | Reference Implementation |
| MCP tool model | Implemented | Uses a minimal read-only tool set that stays portable across hosts | Reference Implementation |
| Skills packaging | Implemented | Uses Agent Skills format with cross-host install targets and trigger descriptions | Reference Implementation |
| Rules/instruction injection | Implemented | Shows both file-based and append-marker insertion models for host variability | Reference Implementation |
| Host setup automation | Implemented | Detects host footprints and performs idempotent config merges | Reference Implementation |
| Hook lifecycle callbacks | Absent | Important caveat: policy and MCP surfaces can exist without runtime pre/post tool hooks | Reference Implementation |
| Version-pinned docs lookup | Implemented | Provides stable output contracts for teams that pin framework versions | Reference Implementation |

Notes:

- Treat this profile as a multi-host integration pattern, not as a full hook-capable lifecycle model.
- Keep fallback guidance explicit when a host or profile lacks hook surfaces.
- Marketplace discovery coverage in this manual is host-agnostic guidance and does not imply host-native install parity.

## Versioning policy for this manual

- Update this matrix whenever vendor docs or implementation-derived behavior changes integration assumptions.
- Keep one row per framework profile, not per provider model.
- For breaking changes, add an entry in docs maintenance changelog.
