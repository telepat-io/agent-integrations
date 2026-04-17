# Transport and Auth Matrix

Use this matrix to choose transport and authentication defaults per host profile.

## Decision matrix

| Host profile | Preferred transport | Typical auth model | When to use |
| --- | --- | --- | --- |
| Local desktop host | stdio | local trust boundary or API key in host config | single-user local workflows |
| Local CLI host | stdio | environment/API key | deterministic local automation |
| Remote team service | HTTPS | API key or OAuth2 | shared infrastructure and access control |
| Enterprise managed environment | HTTPS or mTLS | OAuth2 + policy controls | centralized governance and auditability |

## Practical guidance

- Prefer stdio for local-first setups when host supports process-spawned MCP servers.
- Prefer HTTPS for shared services, centralized auth, and network isolation.
- Keep secrets out of docs; reference env var names and secure stores.
- Document trust boundaries explicitly (local process trust vs remote service trust).

## Auth model checklist

1. Define token source (env var, keychain, vault, OAuth flow).
2. Define rotation path and failure behavior.
3. Define minimal scopes required per tool action.
4. Define revocation response and retry strategy.
5. Define observable error messages for auth failures.

## Verification checks

- Transport works in target host without hidden prompts.
- Auth failures return deterministic, actionable errors.
- Logs do not leak credentials.
- Documentation matches actual config keys and endpoints.
