# daves-digi-doodads - Tracker
## Status: IN PROGRESS
Started: 2026-04-17 | Last Updated: 2026-04-17

## Overview

Public Claude Code plugin marketplace on GitHub. Hosts `goof-proofs`, a plugin bundling Dave's battle-tested skills and rules so friends can bootstrap their own Claude Code setup.

- Marketplace: `github.com/dave-tedder/daves-digi-doodads`
- Plugin: `goof-proofs` (lessons from production)
- Install: `/plugin marketplace add dave-tedder/daves-digi-doodads` then `/plugin install goof-proofs@daves-digi-doodads`

## Todo List

- [x] Task 1 - Scaffold directory structure, init git [Risk: low]
- [ ] Task 2 - Write marketplace.json and plugin.json manifests [Risk: low]
- [ ] Task 3 - Copy + genericize 5 skills (brainstorm, retro, pre-build, systematic-debugging, agent-best-practices) [Risk: low]
- [ ] Task 4 - Copy + genericize ~20 rule files from ~/.claude/rules/ [Risk: medium, content review]
- [ ] Task 5 - Write README.md with install instructions and content overview [Risk: low]
- [ ] Task 6 - Create GitHub repo (public) and push [Risk: low]

## Prohibited Actions

- Never commit secrets, API keys, or personal tokens (check carefully — source rules mention Notion tokens, OpenRouter keys, Supabase URLs)
- Never include Dave's main CLAUDE.md identity block, business context, or tech-stacks.md (contains MCP tokens, account inventory)
- Never include mcp-toggling.md (references personal scripts)
- Never force-push once pushed to origin

## Genericization Rules

- "Dave" → "the user" in skill/rule text that addresses a person
- Real session refs (e.g., "Tedder Trainer Session 43", "Open Brain Session 45") — keep as concrete illustrations; they're valuable as production-lesson signal
- Tool-specific rules stay tool-specific (Supabase, Next.js, Railway) — friends using those stacks will benefit; others skip
- Ship banned-words list empty as a starter template

## Last Completed

**Session:** 1 (2026-04-17)
**What was done:** Scaffolded directory skeleton, init'd git on main
**Next task:** Task 2 - manifests
