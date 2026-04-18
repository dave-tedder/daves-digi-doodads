# daves-digi-doodads

A public [Claude Code](https://claude.com/claude-code) plugin marketplace. Currently hosts one plugin:

## Plugins

### `goof-proofs` (v0.2.0)

Battle-tested skills and reference rules distilled from real production work. Lessons-learned, not theory.

**v0.2 changes:** 19 of v0.1's rules were converted into 21 auto-triggered skills (two of those rules each split into two more-narrowly-triggered skills). Five rules stay as reference markdown (broad-trigger workflow material that's better cherry-picked than auto-loaded). Net: **26 skills + 5 reference rules**, with token cost on install dropping ~85% versus dumping every rule into context on every session.

#### Skills (auto-activate based on context)

Skills load only their description (~30-50 tokens) into every session. The body loads only when the trigger fires.

**Process / workflow (5):**

| Skill | Triggers on |
|---|---|
| `brainstorm` | Thinking through a problem, decision, or fuzzy idea before building |
| `pre-build` | Any non-trivial implementation — forces a "think before you touch a live system" pass |
| `systematic-debugging` | Any bug, error, or unexpected behavior — root cause before fix, no guessing |
| `agent-best-practices` | Multi-step tasks involving subagents or multi-session work |
| `retro` | End-of-session retrospective — routes learnings into rules/memory |

**Stack-specific gotchas (16):**

| Skill | Triggers on |
|---|---|
| `ai-sdk-provider-defaults` | `@ai-sdk/openai` code against OpenRouter / Groq / Together / etc. — silent `/v1/responses` routing bug |
| `css-hidden-attribute` | CSS classes with `display:` rules, or debugging an element with `hidden` that won't hide |
| `git-commit-splitting` | Splitting one file's changes across multiple commits without `git add -p` (no interactive stdin) |
| `icloud-code-in-markdown` | Markdown docs on iCloud Drive with substantial fenced code blocks (which can vanish) |
| `icloud-git-fragility` | Git repos on iCloud Drive — corruption recovery, sibling-worktree grep noise |
| `ios-print-gotchas` | Web app print rendering on iOS Safari — `afterprint` timing, fixed-position trap |
| `ios-safari-inputs` | Mobile iOS web inputs — focus zoom, accessory bar, `dvh` viewport for keyboard pinning |
| `nextjs-middleware-auth` | Next.js App Router auth middleware — 3 silent fail-open modes |
| `nextjs-server-component-events` | Server component passing event handlers to children — build passes, runtime dies |
| `nextjs-server-component-verification` | Verifying a server component or Server Action from outside the browser |
| `node-dev-servers` | `concurrently` failing in Claude Code's background sandbox; CJS / ESM interop in Node 22+ |
| `notion-large-pages` | `notion_retrieve_block_children` exceeding token limits |
| `openrouter-slug-format` | `anthropic/...` model slug to OpenRouter — dot vs dash silent failure |
| `remote-asset-integrity` | Pipelines uploading to R2 / S3 / CDN then triggering processing |
| `supabase-deploy-from-worktree` | Supabase Edge Function deploy from a feature branch worktree |
| `supabase-query-patterns` | supabase-js queries — FK-alias avoidance, `.select('id').single()`, JSONB `.contains()` |

**LLM engineering (2):**

| Skill | Triggers on |
|---|---|
| `llm-decision-scoping` | Designing LLM calls that pick matches from a list — scope candidates before asking |
| `llm-metadata-extraction` | LLM prompts extracting structured metadata from narrative text |

**Computer-use / preview (2):**

| Skill | Triggers on |
|---|---|
| `computer-use-tiers` | Picking between computer-use, Claude-in-Chrome, and Control Chrome MCPs |
| `preview-eval-timeouts` | `preview_eval` timing out at 30s — including write-mutation recovery procedure |

**API odds and ends (1):**

| Skill | Triggers on |
|---|---|
| `youtube-api-gotchas` | YouTube Data API — wrong channel matches, broken-unicode crashes downstream |

#### Reference rules (5 markdown files under `plugins/goof-proofs/rules/`)

These stay as reference rather than skills because their triggers fire on too much of a typical day's work, or because the content is a one-time lookup checklist. Cherry-pick what fits and copy into your `~/.claude/rules/` or paste into your `~/.claude/CLAUDE.md`.

| Rule | Why reference, not skill |
|---|---|
| `apple-health-no-rest-api.md` | Pure fact lookup — no API exists, here's the workaround |
| `railway-domains.md` | Three-step domain setup checklist (CNAME + TXT + port) |
| `security-baseline.md` | General Supabase + Railway hygiene — broad audience, mostly already-known practices |
| `tracking-and-verification.md` | Personal workflow file format (PROJECT-TRACKER + SESSION-LOG) — opinionated, adopt or replace |
| `multi-session-workflow.md` | Personal session-handoff conventions — opinionated, adopt or replace |

## Stacks this is built around

These rules come from one person's production work. The author ships with:

- **Supabase** (Postgres + RLS + Edge Functions + Storage) as the primary backend. Fastest path from idea to production for a small team. RLS as a first-class primitive removes a whole class of auth bugs. Edge Functions put async/scheduled work next to the data.
- **Next.js (App Router)** for frontends. Server components reduce roundtrips, middleware handles auth at the edge, deploys cleanly to Railway.
- **Railway** for hosting. GitHub-triggered deploys, reasonable pricing, managed Postgres when Supabase isn't the right fit.
- **Claude Code + Anthropic SDK + OpenRouter** for LLM work. Claude for capability; OpenRouter for access to other models where they're cheaper or more appropriate (gpt-4o-mini for structured extraction, text-embedding-3-small for vectors).
- **WordPress** for three owned content properties, driven via [AI Engine](https://meowapps.com/plugin/ai-engine/) MCP servers.
- **Notion** for project management, knowledge base, and CRM, driven via Notion's MCP integration.
- **macOS + iOS** — native development environment, with iOS Safari as the primary mobile target and iCloud Drive as the default sync layer (which is where most of the iCloud-specific traps come from).

The skills reflect what hits the edges of those tools. A different stack means many skills simply won't fire (their triggers are narrow and stack-specific). Cross-cutting skills (LLM prompting, computer-use MCP, git commit splitting, the iCloud + git ones for any Mac user) still apply regardless of backend choice.

## Install

```bash
# One-time: add the marketplace
/plugin marketplace add dave-tedder/daves-digi-doodads

# Install the plugin
/plugin install goof-proofs@daves-digi-doodads
```

After install, all 26 skills auto-discover and trigger when their conditions match. Reference rules sit in `plugins/goof-proofs/rules/` (browse on GitHub, or in your local cache at `~/.claude/plugins/cache/`).

## Optional: elevate the 3 "discipline" rules to auto-load

The plugin ships 5 reference rules in `plugins/goof-proofs/rules/`. They sit there as text files — Claude can read them on request, but they don't auto-influence behavior the way skills do.

Three of them I run as auto-loaded discipline in every session, and recommend you do too:

- **`security-baseline.md`** — Supabase + Railway hygiene checklist (the one signal worth the auto-load is the "always destructure `{ error }` from Supabase writes" rule)
- **`tracking-and-verification.md`** — `PROJECT-TRACKER.md` + `SESSION-LOG.md` workflow + verification discipline ("file parses" is not verification)
- **`multi-session-workflow.md`** — handoff format for splitting work across multiple chats

To make them auto-load on every Claude Code session, paste this into your terminal:

```bash
mkdir -p ~/.claude/rules
curl -o ~/.claude/rules/security-baseline.md https://raw.githubusercontent.com/dave-tedder/daves-digi-doodads/main/plugins/goof-proofs/rules/security-baseline.md
curl -o ~/.claude/rules/tracking-and-verification.md https://raw.githubusercontent.com/dave-tedder/daves-digi-doodads/main/plugins/goof-proofs/rules/tracking-and-verification.md
curl -o ~/.claude/rules/multi-session-workflow.md https://raw.githubusercontent.com/dave-tedder/daves-digi-doodads/main/plugins/goof-proofs/rules/multi-session-workflow.md
```

Tradeoff: each rule adds ~250-1,200 words to your session-start context (auto-loaded into every conversation). If you don't use Supabase or Railway, skip `security-baseline`. If you don't run multi-session tracked projects, skip `tracking-and-verification` and `multi-session-workflow`.

Note that `apple-health-no-rest-api.md` and `railway-domains.md` are also shipped as reference rules but I don't recommend auto-loading them — they're pure lookup material (you consult once per Apple Health integration decision or per Railway domain setup, not per session). Read them from the plugin cache when needed: `cat ~/.claude/plugins/cache/daves-digi-doodads/goof-proofs/0.2.0/rules/<filename>.md`.

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

If you spot a genuine bug in one of the skills or rules (e.g. "this claim about Next.js middleware is wrong as of v15.4"), PRs welcome. Stack-preference disagreements ("I'd use Fastify instead of Express") are off-topic — fork.

## Cloning locally

If you clone this repo to read or modify it, do **not** put it in an iCloud Drive folder. Git + iCloud sync causes `.git/objects` corruption. See [`plugins/goof-proofs/skills/icloud-git-fragility/SKILL.md`](plugins/goof-proofs/skills/icloud-git-fragility/SKILL.md) for details. `~/Projects/`, `~/code/`, or anywhere outside iCloud is fine.

## Meta: how these came to exist

Most of these files were written the day after a production incident — literally "here's what just burned us, here's how I'll recognize this next time." The real-world-use paragraphs at the bottom of many skills point to a specific commit or session where the gotcha bit. That context is kept intentionally: reading "Session 45 spent 30 minutes chasing this" communicates something different than "this can happen."

Credit where due: the structure (skills + reference rules, session logs, tracker files) grew from working with Claude Code across multiple projects and noticing what worked. Not claiming invention, just sharing what stuck.
