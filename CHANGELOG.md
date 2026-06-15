# Changelog

## v0.3.1 - 2026-06-15

- Reworded the README opening so the repo reads as a cross-tool starter kit first and a Claude Code plugin marketplace second.
- Added a README contributors section naming Dave Tedder and Codex.
- Updated the `goof-proofs` manifest description to emphasize portable agent skills and Codex-friendly cherry-picking.

## v0.3.0 - 2026-06-15

- Turned the repo into a starter kit, not only a Claude Code plugin marketplace.
- Added `START-HERE.md` as the friend-facing front door.
- Added sanitized global and project templates for Claude Code, Codex, and similar agent tools.
- Added cross-tool setup guides for Claude Code, Codex, Antigravity, and cherry-pick installs.
- Added a project `CLAUDE.md` wrapper template that imports shared `AGENTS.md` guidance for Claude Code users.
- Added 3 public-safe skill authoring helpers: `gh-username-verification`, `skill-authoring-patterns`, and `skill-vs-reference-triage`.
- Bumped `goof-proofs` from 26 skills to 29 skills, with 5 reference rules unchanged.
- Re-ran manifest, skill frontmatter, count, credential, and private-data scans.

## v0.2.2 - 2026-04-18

- Rewrote the `retro` skill invocation guidance.
- Marked `V0.2-TRIAGE.md` as superseded where final v0.2 decisions changed.
- Fixed genericization leaks in `multi-session-workflow.md`.
- Polished README count wording for the v0.2 rules-to-skills split.
- Added the MIT license.
- Bumped `goof-proofs` to `0.2.2`.

## v0.2.1 - 2026-04-18

- Added README guidance for elevating selected reference rules into auto-loaded Claude Code rules.

## v0.2.0 - 2026-04-18

- Converted the bulk of v0.1 reference rules into triggered skills.
- Shipped 26 skills and 5 reference rules.
- Reduced always-loaded personal setup context by moving narrow guidance into skill triggers.

## v0.1.0 - 2026-04-17

- Created the public marketplace and `goof-proofs` plugin.
- Added initial skills, reference rules, plugin manifests, and README install instructions.
