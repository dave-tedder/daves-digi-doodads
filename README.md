# daves-digi-doodads

A public [Claude Code](https://claude.com/claude-code) plugin marketplace. Currently hosts one plugin:

## Plugins

### `goof-proofs`

Battle-tested skills and reference rules distilled from real production work. Lessons-learned, not theory.

**Skills (auto-activate based on context):**

| Skill | Triggers on |
|---|---|
| `brainstorm` | Thinking through a problem, decision, or fuzzy idea before building |
| `pre-build` | Any non-trivial implementation — forces a "think before you touch a live system" pass |
| `systematic-debugging` | Any bug, error, or unexpected behavior — root cause before fix, no guessing |
| `agent-best-practices` | Multi-step tasks involving subagents or multi-session work |
| `retro` | End-of-session retrospective — routes learnings into rules/memory |

**Reference rules (24 markdown files under `plugins/goof-proofs/rules/`):**

Rules are shipped as reference material, not auto-injected. Cherry-pick what fits your stack and drop copies into your own `~/.claude/rules/` directory, or append the relevant ones to your `~/.claude/CLAUDE.md`.

Topics covered:

- **Supabase** — query patterns, deploy-from-worktree gotchas, `.insert/.update` error handling, JSONB patterns
- **Next.js** — middleware auth traps (3 silent-failure modes), server components + event handler traps, server-action curl verification
- **Railway** — custom domains (CNAME + TXT + port), deploy discipline
- **iOS + Safari** — print rendering gotchas, input focus-zoom, accessory bar, `dvh` viewport
- **iCloud + git** — `.git/objects` corruption, concurrent-session hazards, sibling worktree grep noise
- **LLM prompting** — decision-scoping before asking a picker LLM, metadata extraction prompts, hallucinated tag fields, past-tense-to-imperative conversion bugs
- **AI SDK / OpenRouter** — `provider(id)` vs `provider.chat(id)` silent failures, model slug format (dot vs dash)
- **Computer-use MCP** — app tier awareness (read/click/full), Chrome vs Brave, preview_eval 30s timeout, write-mutation recovery procedure
- **Workflow discipline** — tracking-and-verification protocol (PROJECT-TRACKER.md + SESSION-LOG.md), multi-session handoffs, git commit splitting
- **Security baseline** — Supabase RLS, Railway env var hygiene, never-log-PII
- **Platform odds and ends** — YouTube API gotchas, Notion large page retrieval, Apple Health no-REST, remote asset integrity

## Install

```bash
# One-time: add the marketplace
/plugin marketplace add davetedder/daves-digi-doodads

# Install the plugin
/plugin install goof-proofs@daves-digi-doodads
```

After install, the 5 skills are immediately available. Claude Code auto-discovers them and invokes them when their trigger conditions match. To use the reference rules, browse `plugins/goof-proofs/rules/` on GitHub (or in your local plugin cache at `~/.claude/plugins/cache/`) and copy what applies to your stack.

## Updating

```bash
/plugin marketplace update daves-digi-doodads
```

## Uninstall

```bash
/plugin uninstall goof-proofs
/plugin marketplace remove daves-digi-doodads
```

## Forking / contributing

This is a personal knowledge dump made public so friends can bootstrap. If you want your own version, fork the repo, replace the marketplace name in `.claude-plugin/marketplace.json`, add your own plugins under `plugins/`, and point your friends at your fork.

If you spot a genuine bug in one of the rules (e.g. "this claim about Next.js middleware is wrong as of v15.4"), PRs welcome. Stack-preference disagreements ("I'd use Fastify instead of Express") are off-topic — fork.

## Cloning locally

If you clone this repo to read or modify it, do **not** put it in an iCloud Drive folder. Git + iCloud sync causes `.git/objects` corruption. See [`plugins/goof-proofs/rules/icloud-git-fragility.md`](plugins/goof-proofs/rules/icloud-git-fragility.md) for details. `~/Projects/`, `~/code/`, or anywhere outside iCloud is fine.

## Meta: how these rules came to exist

Most of these files were written the day after a production incident — literally "here's what just burned us, here's how I'll recognize this next time." The real-world-use paragraphs at the bottom of many rules point to a specific commit or session where the gotcha bit. That context is kept intentionally: reading "Session 45 spent 30 minutes chasing this" communicates something different than "this can happen."

Credit where due: the structure (rules in `~/.claude/rules/`, session logs, tracker files) grew from working with Claude Code across multiple projects and noticing what worked. Not claiming invention, just sharing what stuck.
