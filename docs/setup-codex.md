# Codex Setup

Use this path if you want the same workflow habits in Codex without relying on Claude Code's plugin marketplace.

## Global Instructions

Codex global preferences live here:

```text
~/.codex/AGENTS.md
```

Start from:

```text
templates/global/AGENTS.md
```

Review the template before copying it. Add only preferences that should apply everywhere.

## Project Instructions

Project-specific Codex guidance lives in an `AGENTS.md` file at the project root.

Start from:

```text
templates/project/AGENTS.md
```

Fill in the project's real commands, structure, and guardrails. Keep secrets out.

## Skills

Codex skills can live under:

```text
~/.codex/skills/<skill-name>/SKILL.md
```

To copy a selected skill from this repo:

```bash
mkdir -p ~/.codex/skills/systematic-debugging
cp plugins/goof-proofs/skills/systematic-debugging/SKILL.md ~/.codex/skills/systematic-debugging/SKILL.md
```

Repeat that only for skills you want available in Codex. The portable unit is the whole skill folder, especially if a skill later gains scripts, examples, or templates.

## Project Tracking

For tracked project work, copy:

```text
templates/project/PROJECT-TRACKER.md
templates/project/SESSION-LOG.md
```

Then tell future agents to read the tracker first and the newest session log entry second.

## Reference Rules

The markdown files under `plugins/goof-proofs/rules/` are not Codex skills. Treat them as reference material you can adapt into:

- `~/.codex/AGENTS.md`
- a project `AGENTS.md`
- a project-specific doc under `docs/`

Keep them narrow. Global instructions should be for habits you really want in every session.
