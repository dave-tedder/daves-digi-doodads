# daves-digi-doodads - Tracker
## Status: PLANNED (v0.3 starter kit)
Started: 2026-04-17 | Last Updated: 2026-06-15 | Live: https://github.com/dave-tedder/daves-digi-doodads

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
- [x] Task 9 - v0.2.2: retro skill — rewrite "When to Use" to cover both slash invocation + natural language [Risk: low]
- [x] Task 10 - v0.2.2: V0.2-TRIAGE.md — add superseded note at top pointing to Session 2 flips [Risk: low]
- [x] Task 11 - v0.2.2: multi-session-workflow.md — genericize 2 pronoun leaks + drop dangling tech-stacks.md ref [Risk: low]
- [x] Task 12 - v0.2.2: README.md — polish rule/skill count phrasing to show the 19→21 split math [Risk: low]
- [x] Task 13 - v0.2.2: Add LICENSE (MIT) [Risk: low]
- [x] Task 14 - v0.2.2: Bump plugin.json to 0.2.2, close session, tag [Risk: low]
- [x] Task 15 - v0.3 starter kit planning: save implementation plan and short kickoff prompt in `docs/plans/2026-06-15-v0.3-starter-kit.md` [Risk: low]
- [ ] Task 16 - v0.3 starter kit execution: add friend-facing docs, sanitized templates, cherry-pick recipes, and cross-tool setup guidance [Risk: medium, content review — must avoid sensitive/private config]

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

**Session:** 4 (2026-06-15) — PLANNING COMPLETE
**What was done:** Saved the full v0.3 starter-kit execution plan at `docs/plans/2026-06-15-v0.3-starter-kit.md`. The plan keeps `goof-proofs` as the full bundle while adding friend-facing docs, sanitized templates, cherry-pick recipes, and cross-tool setup guidance for Claude Code, Codex, Antigravity, and similar agent tools.
**Next task:** Execute Task 16 from a fresh session using `docs/plans/2026-06-15-v0.3-starter-kit.md`. Start by reading this tracker, then the latest `SESSION-LOG.md` entry, then create `codex/v0.3-starter-kit` unless it already exists.
