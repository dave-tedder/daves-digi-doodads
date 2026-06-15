# Antigravity And Other Agent Tools

Use this repo as a source library when your agent tool does not understand Claude Code marketplaces.

## Portable Pieces

The most portable pieces are:

- `plugins/goof-proofs/skills/<skill-name>/SKILL.md`
- `plugins/goof-proofs/rules/*.md`
- `templates/global/`
- `templates/project/`

Copy the pieces your tool can read, then adapt filenames and locations to that tool's instruction system.

## Skills

Each skill is a folder with a `SKILL.md` file. Preserve that folder shape when your tool supports skill-style loading.

If your tool only supports plain instruction files, paste the parts that matter into a project instruction file instead of copying every skill globally.

## Rules And Preferences

The `rules/` files are reference guidance. They are useful when you want a checklist or operating rule, but they may be too broad to load into every conversation.

Start with:

- `templates/global/AGENTS.md` for global preferences
- `templates/project/AGENTS.md` for project-specific guidance
- `templates/project/PROJECT-TRACKER.md` and `templates/project/SESSION-LOG.md` for continuity

## Claude Plugin Commands

Claude Code plugin commands such as `/plugin marketplace add` and `/plugin install` are for Claude Code. They are not the install path for Antigravity or other tools.

For other tools, clone or download this repo and copy the relevant files manually.

## Safety Check

Before copying any instruction file from any person or repo, review it for:

- private identity blocks
- API keys or tokens
- private URLs
- account inventory
- business-only configuration
- tool rules that do not fit your workflow
