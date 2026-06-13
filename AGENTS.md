# daves-digi-doodads

Public plugin marketplace containing the `goof-proofs` plugin and Dave's genericized production lessons.

## Startup

Read `PROJECT-TRACKER.md`, then the newest `SESSION-LOG.md` entry. Check `git status --short --branch` before editing. Use `README.md` for the public contract and `V0.2-TRIAGE.md` only as a historical artifact; its superseded note points to the final decisions.

## Structure

- `.claude-plugin/marketplace.json`: marketplace manifest
- `plugins/goof-proofs/.claude-plugin/plugin.json`: plugin manifest and version
- `plugins/goof-proofs/skills/`: triggered skills
- `plugins/goof-proofs/rules/`: reference guidance

## Verification

- Parse both JSON manifests.
- Confirm every skill has valid YAML frontmatter and a narrow trigger.
- Re-run secret and personal-data scans before publishing.
- Verify README counts match the actual skill and rule directories.

## Guardrails

- Never publish secrets, tokens, personal account inventories, Dave's identity block, or business-only configuration.
- Genericize reusable guidance to "the user" while preserving useful technical examples.
- Do not force-push published history.
- Keep version bumps, documentation, and tags aligned.
