# daves-digi-doodads - Tracker
## Status: COMPLETE (through v0.2.1)
Started: 2026-04-17 | Last Updated: 2026-04-18 | Live: https://github.com/dave-tedder/daves-digi-doodads

## Overview

Public Claude Code plugin marketplace on GitHub. Hosts `goof-proofs`, a plugin bundling Dave's battle-tested skills and rules so friends can bootstrap their own Claude Code setup.

- Marketplace: `github.com/dave-tedder/daves-digi-doodads`
- Plugin: `goof-proofs` (lessons from production)
- Install: `/plugin marketplace add dave-tedder/daves-digi-doodads` then `/plugin install goof-proofs@daves-digi-doodads`

## Todo List

- [x] Task 1 - Scaffold directory structure, init git [Risk: low]
- [x] Task 2 - Write marketplace.json and plugin.json manifests [Risk: low]
- [x] Task 3 - Copy + genericize 5 skills [Risk: low]
- [x] Task 4 - Copy + genericize 24 rule files from ~/.claude/rules/ [Risk: medium, content review — secret scan passed, no leaks]
- [x] Task 5 - Write README.md with install + stacks-used + forking notes [Risk: low]
- [x] Task 6 - Create public GitHub repo and push [Risk: low]
- [x] Task 7 - v0.2 skills refactor: 21 rules → skills (with 2 splits + 2 consolidations), 5 stay as rules, deploy to personal ~/.claude/, tag v0.2.0 [Risk: medium — touches personal setup]
- [x] Task 8 - v0.2.1 README addition: explain how friends elevate 3 reference rules to auto-load via curl

## Follow-ups (future sessions)

- **v0.2 refactor — triage done, ready for execution session.** See `V0.2-TRIAGE.md` in this directory. 22 rules convert to 24 skills (2 split), 2 rules stay as reference. Expected: dual win of better plugin UX + personal token load drop. Estimate: 3-4 hours focused across 1-2 sessions.
- Open-source Open Brain as a separate repo — retro skill and ai-sdk-openai-provider-defaults rule reference `github.com/dave-tedder/open-brain` which doesn't exist yet. Rules still work as narrative; the link 404s until Open Brain is published.
- Consider adding a `playbooks/` directory to the plugin later for opinionated multi-step workflows (meal-plan-style aggregation, end-of-session retro, etc.) beyond single-skill triggers.
- If friends adopt it, collect their first-week "what confused me" feedback and fold into README.

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

**Session:** 2 (2026-04-18) — COMPLETE
**What was done:** v0.2.0 shipped (21 rules → 21 skills with 2 splits + 2 consolidations; 5 reference rules stay). Plugin merged to main, tagged v0.2.0, pushed. Then v0.2.1 patch added a README section explaining how friends elevate the 3 behavioral reference rules (security-baseline, tracking-and-verification, multi-session-workflow) to auto-load via 3 curl commands. Plugin installed locally; 21 redundant rules + 2 lookup rules + 5 v0.1 skills deleted from personal `~/.claude/rules/` and `~/.claude/skills/`. Personal setup now auto-loads 10 rules + plugin skill metadata. Actual token reduction: ~22K → ~10.8K tokens per cold session (~51% reduction). Less than the projected 67% because Dave kept 3 behavioral rules auto-loading (correct call).
**Next task:** none — project complete through v0.2.1. Future work:
- Watch for over-trigger reports on individual skills in real use
- Consider v0.3 if friends adopt and surface gaps
