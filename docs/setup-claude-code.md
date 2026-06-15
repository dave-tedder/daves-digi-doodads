# Claude Code Setup

Use this path if you want the full `goof-proofs` bundle through Claude Code's plugin system.

## Install The Marketplace

In Claude Code, run:

```bash
/plugin marketplace add dave-tedder/daves-digi-doodads
```

Then install the plugin:

```bash
/plugin install goof-proofs@daves-digi-doodads
```

After install, restart or reload plugins if Claude Code asks you to.

## What Loads Automatically

The `skills/` folders auto-discover. Claude Code sees each skill's frontmatter description, then loads the full `SKILL.md` body only when the trigger matches the task.

The markdown files under `plugins/goof-proofs/rules/` are reference files. They do not auto-load just because the plugin is installed.

## Browse Installed Files

Claude Code caches installed plugin files under:

```text
~/.claude/plugins/cache/daves-digi-doodads/goof-proofs/<installed-version>/
```

Useful paths:

```text
skills/<skill-name>/SKILL.md
rules/<filename>.md
```

## Optional Reference Rules

If you want a reference rule to affect every Claude Code session, copy it into your global rules or project instructions.

Example:

```bash
mkdir -p ~/.claude/rules
curl -o ~/.claude/rules/tracking-and-verification.md https://raw.githubusercontent.com/dave-tedder/daves-digi-doodads/main/plugins/goof-proofs/rules/tracking-and-verification.md
```

Do this only for rules you actually want in every session. Auto-loaded rules spend context every time.

## Templates

Use these as starting points:

- `templates/global/CLAUDE.md` for global Claude Code preferences
- `templates/project/AGENTS.md` for project-local startup and guardrails
- `templates/project/PROJECT-TRACKER.md` and `templates/project/SESSION-LOG.md` for tracked project work
