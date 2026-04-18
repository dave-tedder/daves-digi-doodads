# daves-digi-doodads - Session Log

## Session Index
<!-- One line per session. Newest at top. Format: Session N (date) - [summary] -->
- Session 2 (2026-04-18) - v0.2 skills refactor: 21 rules → skills, 5 stay as rules, deploy + tag v0.2.0
- Session 1 (2026-04-17) - Initial scaffolding, manifests, skills, rules, README, push

## Session 2 - 2026-04-18

**Task(s) in progress:** v0.2 skills refactor per V0.2-TRIAGE.md.

**Triage flips before execution (judgment-call review with Dave):**
- `tracking-and-verification` — flipped from skill back to rule. Trigger "any task spanning multiple steps" over-fires. Friends already have superpowers' verification-before-completion. Workflow file format is reference material, read once.
- `security-baseline` — flipped from skill back to rule. Trigger "auth/db/secrets code" matches every backend session. Most items are general hygiene. Add a framing note that the unique signal is the Supabase write-error-destructure pattern.
- `multi-session-workflow` — flipped from skill back to rule. Same broad workflow trigger as tracking-and-verification. Friends will have their own handoff conventions.

**Consolidations during conversion:**
- `supabase-query-patterns` skill drops the "always destructure { error }" paragraph (already lives in `security-baseline` rule). Keeps FK-alias avoidance, `.select('id').single()` after insert, JSONB `.contains()`, non-empty array filter.
- `computer-use-tiers` skill trimmed to Dave-specific lessons not already in the computer-use MCP server's auto-injected instructions. Keeps Chrome vs Brave, Claude-in-Chrome tab groups + login state, Cowork agent fallbacks, `form_input` vs `type` on claude.ai, iframe limitations, Control Chrome reliability. Drops the tier explanation since the MCP injection already covers it.

**Net plan:** 21 skills + 5 reference rules.
- Skills (21): ai-sdk-provider-defaults, css-hidden-attribute, git-commit-splitting, icloud-code-in-markdown, icloud-git-fragility, ios-print-gotchas, ios-safari-inputs, llm-decision-scoping, llm-metadata-extraction, nextjs-middleware-auth, nextjs-server-component-events, nextjs-server-component-verification, node-dev-servers, notion-large-pages, openrouter-slug-format, remote-asset-integrity, supabase-deploy-from-worktree, supabase-query-patterns, youtube-api-gotchas, computer-use-tiers, preview-eval-timeouts
- Rules (5): apple-health-no-rest-api, railway-domains, tracking-and-verification, security-baseline, multi-session-workflow

**What was done:** _(filled in as session progresses)_

**Files modified:** _(filled in)_

**Verification:** _(filled in)_

**Commit(s):** _(filled in)_

**Notes:** _(filled in)_

## Session 1 - 2026-04-17

**Task(s) completed:** Tasks 1-6, all six. Project shipped end-to-end.

**What was done:**
- Confirmed plugin spec via claude-code-guide agent: marketplace needs `.claude-plugin/marketplace.json` at repo root; plugin lives at `plugins/goof-proofs/` with its own `.claude-plugin/plugin.json`; skills auto-discovered from `skills/<name>/SKILL.md`.
- Key finding: rules markdown files don't auto-inject via the plugin system. Shipped them as reference files in `rules/` subdir with README instructions for friends to cherry-pick.
- 5 skills genericized: replaced "Dave" with "the user" throughout; swapped Dave-specific stack mentions for stack-agnostic framings in pre-build; reframed Open Brain references in retro as "your memory system (e.g., Open Brain)".
- 24 rules copied and genericized. Excluded `mcp-toggling.md` (references personal scripts) and `tech-stacks.md` (contains live Notion API token, WordPress MCP URL secrets, Supabase URL with key).
- Real-world incident references (Open Brain Session N, Tedder Trainer Session N) kept per user direction — zero security/embarrassing content, and they provide production-signal illustrations that make rules memorable.
- Pre-push secret scan: zero hits on token/key patterns across all shipped files.
- Caught username bug before push: authenticated `gh` account is `dave-tedder` (hyphen), not `davetedder` as user originally stated. Corrected across 5 files.
- Added "Stacks this is built around" section to README per user request so readers can judge relevance before installing.
- Created public repo `github.com/dave-tedder/daves-digi-doodads` via `gh repo create --public --source=. --push`.

**Files modified:**
- `PROJECT-TRACKER.md`, `SESSION-LOG.md` (new, tracking)
- `.claude-plugin/marketplace.json` (new)
- `plugins/goof-proofs/.claude-plugin/plugin.json` (new)
- `plugins/goof-proofs/skills/{agent-best-practices,brainstorm,pre-build,retro,systematic-debugging}/SKILL.md` (new × 5)
- `plugins/goof-proofs/rules/*.md` (new × 24)
- `README.md` (new)

**Verification:**
- `git log --oneline` shows 7 commits (Tasks 1-5 + username fix + this close-out pending).
- `gh repo create` returned 200 with the live URL.
- `git push` set upstream tracking to `origin/main`.
- Secret scan: 0 hits on `sk-`, `ntn_`, `sbp_`, `xoxb-`, `ghp_`, `gho_`, `AKIA`, and other common credential prefixes across the plugin directory.
- Personal-name scan: 0 hits on "Dave" (post-genericization) outside legitimate `davetedder.com` domain references and `dave-tedder/open-brain-dashboard` GitHub repo references.

**Commit(s):**
- `bbf00ff` - Task 1: Scaffold project tracker and session log
- `6c67751` - Task 2: Add marketplace and plugin manifests
- `4e76585` - Task 3: Add 5 skills
- `688e966` - Task 4: Add 24 genericized reference rules
- `4a72378` - Task 5: Add README with install instructions and content overview
- `da4b6b0` - Fix GitHub username (dave-tedder) and add stacks-used section to README
- (this commit) - Task 6 close-out: tracker/session-log finalization

**Notes:**
- **iCloud location** chosen by user despite own rule flagging iCloud+git fragility. README warns future cloners to clone off iCloud. User's own repo copy is on iCloud — monitor for the `.git/objects` corruption symptom if doing extensive further work here; if it happens, re-clone from origin.
- **Open Brain link is forward-referenced** — `retro/SKILL.md` and `ai-sdk-openai-provider-defaults.md` link to `github.com/dave-tedder/open-brain` which doesn't exist yet. Rules still stand as narrative. When user open-sources Open Brain, the links will resolve.
- **Plugin install caching:** plugins are copied to `~/.claude/plugins/cache/`. All files must live inside the plugin dir (external refs won't resolve post-install). Verified tree has no external refs.
- **Pre-build skill** intentionally kept "WordPress, Notion, Supabase, Railway, Claude Code" as example stack elements in one place (as representative examples, not declarations). Generic enough for friends' reading; concrete enough to be useful. If a friend's stack is completely different they'll adjust mentally.
- **Marketplace vs plugin install commands** — README shows `/plugin marketplace add dave-tedder/daves-digi-doodads` then `/plugin install goof-proofs@daves-digi-doodads`. The `@daves-digi-doodads` is the marketplace alias, not the owner.
