# Skills Marketplace Integration

## Objective

Add a neutral, repeatable process for discovering and vetting third-party skills from community directories such as SkillsMP, and include the native OpenClaw registry path through ClawHub.

## Positioning

Treat SkillsMP as a discovery directory, not as an authority or trust boundary.

- Use directories to find candidate skills faster.
- Keep installation, security review, and compatibility checks in your own workflow.
- Avoid language that implies automatic trust or guaranteed quality.

For OpenClaw-specific integrations, ClawHub is the primary native registry path and should be documented first for install/update flows.

## OpenClaw and ClawHub quick path

For OpenClaw users, prefer native `openclaw` commands for day-to-day install and update:

```bash
openclaw skills search "calendar"
openclaw skills install <skill-slug>
openclaw skills update --all
openclaw plugins install clawhub:<package>
openclaw plugins update --all
```

Use the `clawhub` CLI for registry-authenticated workflows such as publish and sync:

```bash
clawhub login
clawhub search "postgres backups"
clawhub install <skill-slug>
clawhub update --all
clawhub sync --all
```

Operational guidance:

- Standardize on one install/update flow per environment (native `openclaw` or `clawhub`) to reduce lock/source confusion.
- Start a new session after major skill changes so updated snapshots are visible immediately.

## When marketplace discovery is useful

Use marketplace discovery when:

- your team needs faster discovery across many skill repositories,
- you want richer search filters (keyword, category, occupation),
- you need repeatable intake pipelines for skill curation.

Skip marketplace-first discovery when:

- you only use a small, internal allowlist,
- your workflow requires strict provenance from known authors only,
- tasks are one-off and do not justify a reusable skill.

## Recommended intake workflow

1. Discover
- Search by task intent and platform compatibility.
- Record candidate source repository URLs and commit activity.

2. Evaluate
- Check maintainer activity and release cadence.
- Check clarity of SKILL.md, references, and assets.
- Check whether expected outputs and trigger conditions are explicit.

3. Security review
- Treat skills as open-source code.
- Review scripts and external command usage before install.
- Prefer trusted maintainers and explicit licenses.
- Do not install unreviewed skills into production profiles.

4. Install and verify
- Install into the correct host scope (project or global).
- Run verification prompts from your required skills matrix.
- Confirm the host actually loads the skill on restart where required.

5. Curate
- Promote approved skills into a team allowlist.
- Record rejection reasons for failed candidates.
- Re-review periodically because upstream content can change.

## Adopt and harden imported skills

After initial intake, normalize third-party skills before broad rollout.

1. Normalize metadata and structure
- Ensure `SKILL.md` frontmatter is spec-compliant (`name`, `description`, optional fields valid).
- Align skill directory naming with frontmatter `name`.
- Add missing `compatibility` notes only when environment requirements are real and specific.

2. Validate trigger quality
- Confirm description clearly states when to trigger and when not to.
- Run a small trigger test set with positive and near-miss negative prompts.
- Reject skills that over-trigger on adjacent tasks.

3. Validate execution quality
- Run with-skill versus baseline tasks for representative prompts.
- Prefer objective assertions for output checks; add human review for subjective quality.
- Track time and token overhead where your host exposes those signals.

4. Review scripts and commands
- Verify all bundled scripts are non-interactive.
- Require clear usage/help output and deterministic exit behavior.
- Pin external tool versions where possible to reduce drift.
- Reject scripts that require broad unsafe permissions without justification.

5. Publish curated profile
- Store approved skill version and source commit/tag.
- Document host scope (repo, user, system) and required restart behavior.
- Link each approved skill to one verification prompt and one rollback path.

## Optional API-assisted discovery

SkillsMP exposes REST endpoints that can support optional automation in curation workflows.

- Keyword search endpoint: GET /api/v1/skills/search
- Semantic search endpoint: GET /api/v1/skills/ai-search
- Common filters: q, page, limit, sortBy, category, occupation
- Authentication: Bearer API key for authenticated limits and AI search
- Quotas: anonymous and authenticated rate limits are different

Implementation guidance:

- Keep API usage optional, never required for baseline setup.
- Cache and review results before suggesting install actions.
- Handle 401, 429, and 5xx responses with clear fallback behavior.

## Evaluation and promotion gates

Use explicit promotion gates before moving marketplace skills into default installs.

- Gate 1: specification compliance
	- frontmatter validity and path structure checks pass.
- Gate 2: trigger correctness
	- acceptable pass rate for should-trigger and should-not-trigger eval sets.
- Gate 3: output quality
	- skill improves key assertions over baseline on representative tasks.
- Gate 4: operational safety
	- scripts and commands meet non-interactive and safe-default requirements.
- Gate 5: portability
	- validated on every host profile where the skill is marked supported.

Do not auto-promote based only on marketplace popularity or star count.

## Skills vs MCP

Skills and MCP should be treated as complementary surfaces.

- Skills: reusable knowledge, workflows, and conventions.
- MCP: runtime tool/function access to external systems.

Use skills to standardize behavior and MCP to execute capability.

## Compatibility caveats

Marketplace listings may include skills that are platform-specific.

- Validate host compatibility tags before install.
- Do not assume Web, Desktop, and CLI parity for every skill.
- For skills that rely on MCP tooling, confirm the target host supports that tooling path.

## Governance recommendations

- Publish a written allowlist policy for approved skill sources.
- Require artifact proof for any "recommended" skill claim.
- Keep a changelog for approved skill updates and removals.
- Keep artifact evidence for every approval decision (eval results, review notes, source revision).

## Related pages

- [Instruction Layers and Skills](instructions-and-skills.md)
- [Required Skills Matrix](../references/required-skills-matrix.md)
- [New CLI Integration Checklist](../references/new-cli-checklist.md)
- [Compatibility Matrix](../references/compatibility-matrix.md)
- [Source and Evidence Matrix](../references/source-matrix.md)

## Official references

- ClawHub: https://docs.openclaw.ai/tools/clawhub
- Skills: https://docs.openclaw.ai/tools/skills
- Skills Config: https://docs.openclaw.ai/tools/skills-config
