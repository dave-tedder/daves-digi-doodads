# daves-digi-doodads - Session Log

## Session Index
<!-- One line per session. Newest at top. Format: Session N (date) - [summary] -->
- Session 3 (2026-04-18) - v0.2.2 polish fixes from full-project review (retro framing, pronoun leaks, LICENSE, README count, V0.2-TRIAGE superseded note)
- Session 2 (2026-04-18) - v0.2.0 skills refactor (21 rules → skills) + v0.2.1 README patch + personal setup deployed, ~51% token reduction
- Session 1 (2026-04-17) - Initial scaffolding, manifests, skills, rules, README, push

## Session 3 - 2026-04-18

**Task(s) in progress:** v0.2.2 polish fixes from full-project review ("ultrareview").

**Context:** User asked for a top-to-bottom review of the project at the v0.2.1 shelf-stable point. Review surfaced 3 real issues and 2 cleanup items. User confirmed all 5 for fixing. One additional genericization bug (pronoun leak in `multi-session-workflow.md`) discovered mid-verification and rolled into this session.

**Findings that became tasks:**

1. **Retro skill misleading invocation text.** [`skills/retro/SKILL.md:14`](plugins/goof-proofs/skills/retro/SKILL.md) said "Call `/retro` when wrapping up a session" — but retro is a skill (auto-triggered), not a slash command. User clarified: in Claude Code it CAN be invoked via `/retro` slash OR via natural language — rewrite to cover both.

2. **V0.2-TRIAGE.md stale counts.** Said "22 rules → 24 skills, total 29." Actual ship was 21 new skills + 5 reference due to 3 judgment-call flips during Session 2 execute. User's call: add superseded note at top (keeps historical artifact value).

3. **Pronoun leaks in multi-session-workflow.md.** Line 96 "what he's trying to accomplish", line 97 "Factor in his available tech stacks (see tech-stacks.md)". Second one also has a dangling reference to `tech-stacks.md` which is NOT shipped in the plugin (it's a personal rule). Genericize both + drop the broken parenthetical.

4. **README count imprecision.** Line 11 said "21 rules from v0.1 became auto-triggered skills" — 21 is the skill count, but only 19 source rules were converted (2 of those split into 2 skills each = 17 + 4 = 21 skills). Polish to make the 19→21 split math explicit.

5. **No LICENSE file.** Public marketplace with no LICENSE defaults to "all rights reserved" on GitHub. "Personal knowledge dump made public so friends can bootstrap" framing implies permissive license. Adding MIT.

**Non-findings (confirmed clean during review):**
- Zero secret leaks (scanned `sk-`, `ntn_`, `sbp_`, `ghp_`, `xoxb-`, `AKIA`, `-----BEGIN`).
- Zero leftover `\bDave\b` / `Tedder` / unhyphenated `davetedder` references outside legit `davetedder.com` domain + `dave-tedder` GitHub handle.
- All 26 skills have valid YAML frontmatter with narrow, situational "Use when X" descriptions.
- Skill sizes reasonable (smallest 16 lines, largest 117 lines, no bloat).
- Counts consistent across README, plugin.json description, and tracker (26 skills + 5 reference rules).
- `retro/SKILL.md:28` links to private `open-brain` repo — user plans to open-source soon, so link self-resolves. Narrative `open-brain-dashboard` references in `ai-sdk-provider-defaults` and `ios-safari-inputs` are plain-text commit refs (not links), read fine without resolvable URL.

**Personal-rule stale count (OUT OF SCOPE for this repo, noting for later):**
- The personal `~/.claude/rules/multi-session-workflow.md` has a "Triage Before Execute" section with "final 19 skills + 5 reference" — the "19" is stale (actual is 21 new skills). Plugin-shipped copy does NOT have this section, so no plugin fix needed. User can update personal rule separately.

**What was done:** (filled in as tasks complete)

**Files modified:**

**Verification:**

**Commit(s):**

**Notes:**

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

**Other-session retro discovery (mid-session):**

Mid-conversion, Dave flagged that a separate chat had finished a retro on the v0.1 ship and added 5 new rules to `~/.claude/rules/`. Reviewed each:
- `claude-code-plugins.md` — plugin structure facts (Jan 2026 snapshot)
- `claude-code-rules-token-cost.md` — measured the very thing Session 2 was about to measure: 26 rules ≈ 22K tokens auto-loaded per session
- `gh-username-verification.md` — captures the davetedder vs dave-tedder bug from Session 1 as a checklist
- `skill-authoring-patterns.md` — the "use 'the user' not name placeholders," "narrow triggers > short bodies," "split mixed rules" guidance
- `skill-vs-reference-triage.md` — the decision framework, references this exact session's flips back to reference

Decision: **don't ship any of these in goof-proofs.** Audience mismatch — goof-proofs is for friends INSTALLING the plugin, not BUILDING plugins. The 5 retro rules guide plugin/skill authoring work which is Dave's personal context. They live at home in `~/.claude/rules/`. Skipping them avoids bloat without losing signal — anyone forking the plugin can read the official Anthropic docs for plugin/skill authoring.

These 5 will survive the upcoming "delete duplicates" deploy because they aren't duplicates of plugin content.

**What was done:**
- Created v0.2-skills-refactor branch.
- Measured BEFORE-state token load on Dave's personal setup.
- Converted 21 rules into skills in 8 commits (logical bundles: iOS, iCloud, LLM, Next.js, Supabase, AI/OpenRouter, misc×2, computer-use).
- 2 splits: nextjs-server-components → events + verification; computer-use-tiers → tiers + preview-eval-timeouts.
- 2 consolidations: stripped duplicate "destructure { error }" paragraph from supabase-query-patterns (lives in security-baseline rule); trimmed computer-use-tiers skill body to drop content already covered by the computer-use MCP server's auto-injected guidance (tier system, browser fallback, request-access flow).
- Renamed `ai-sdk-openai-provider-defaults` → `ai-sdk-provider-defaults` (the rule applies to all OpenAI-compatible providers, not just OpenAI).
- Added top-of-file framing note to `security-baseline.md` highlighting that the unique signal is the Supabase write-error-destructure rule; the rest is general hygiene experienced devs already practice.
- Bumped `plugin.json` to v0.2.0; expanded keywords; rewrote description for the skills-vs-rules split.
- Rewrote README skills section: 26 skills tabled by category (process / stack-specific / LLM / computer-use / API), 5 reference rules with rationale for why each stays as reference. Updated icloud-git-fragility link from old rule path to new skill path.

**Files modified:**
- `PROJECT-TRACKER.md`, `SESSION-LOG.md` (this file)
- `plugins/goof-proofs/skills/<name>/SKILL.md` × 21 new (and dirs)
- `plugins/goof-proofs/rules/*.md` — 19 deleted (the 21 conversions minus the 2 splits which combined 1 rule into 2 skills each, so 21 - 2*1 = 19 rule deletions)
- `plugins/goof-proofs/rules/security-baseline.md` — framing note added
- `plugins/goof-proofs/.claude-plugin/plugin.json` — version bump
- `README.md` — full skills/rules section rewrite

**Verification (BEFORE/AFTER token measurement) — actual post-deploy:**

| Auto-loaded into every session | BEFORE | AFTER (actual) |
|---|---:|---:|
| `~/.claude/CLAUDE.md` | 680 w | 680 w |
| `~/.claude/rules/` | 13,262 w (26 files) | 6,183 w (10 files) |
| `~/.claude/skills/` | 2,305 w (5 dirs) | 0 w (empty) |
| Plugin skill metadata (frontmatter only, body lazy-loaded) | n/a | 1,121 w |
| **Total auto-loaded** | **~16,247 w** | **~7,984 w** |
| **Approximate tokens** | **~22,000** | **~10,800** |

Net: **~51% reduction in always-loaded tokens** (~11,200 tokens saved per cold session). Less than the 67% projection because Dave opted to keep 3 behavioral rules auto-loading (`tracking-and-verification`, `security-baseline`, `multi-session-workflow`) — the correct call given those are real discipline guidance, not just reference. The 26 plugin skill bodies (~12,700 words combined) only enter context when their narrow trigger fires.

Personal `~/.claude/rules/` after cleanup (10 files): `claude-code-plugins`, `claude-code-rules-token-cost`, `gh-username-verification`, `mcp-toggling`, `multi-session-workflow`, `security-baseline`, `skill-authoring-patterns`, `skill-vs-reference-triage`, `tech-stacks`, `tracking-and-verification`. Personal `~/.claude/skills/` empty (5 v0.1 skills now served by plugin).

**Commit(s) on v0.2-skills-refactor branch:**
- `9d9ffb6` - Open Session 2 entry for v0.2 skills refactor
- `d036528` - v0.2: convert iOS rules to skills
- `e4170fb` - v0.2: convert iCloud rules to skills
- `c01233f` - v0.2: convert LLM rules to skills
- `9f3f9c2` - v0.2: convert Next.js rules to skills (3 skills, including 1 split)
- `49de2b5` - v0.2: convert Supabase rules to skills (with error-destructure dedup)
- `ad5a474` - v0.2: convert AI SDK + OpenRouter rules to skills
- `8358d17` - v0.2: convert misc rules to skills (css-hidden, git-commit-splitting)
- `e9e9d39` - v0.2: convert misc bundle 2 to skills (node, notion, asset, youtube)
- `fbaa0b2` - v0.2: convert computer-use rule into 2 skills (with content trim)
- `fcb9230` - v0.2.0: bump plugin version, rewrite README, add framing to security-baseline
- `7abbbf0` - v0.2 close-out: finalize Session 2 entry with measurements + handoff
- `a7900a1` - Merge v0.2-skills-refactor to main (no-ff, preserves branch history)
- `6dafd6f` - v0.2.1: README — explain how to elevate 3 reference rules to auto-load

Tags: `v0.2.0`, `v0.2.1` — both pushed to origin.

**Morning deploy (post-handoff):**
- Dave opened actual Claude Code CLI (`~/.local/bin/claude`, v2.1.89) — confirmed Desktop's Code-tab does NOT support `/plugin` slash commands; only the CLI does.
- `/plugin marketplace update daves-digi-doodads` opened the new TUI plugin browser; install completed via the UI; `/reload-plugins` activated 26 skills + 6 agents + 1 hook.
- Plugin cache landed at `~/.claude/plugins/cache/daves-digi-doodads/goof-proofs/0.2.0/` (then 0.2.1 after the README patch). 26 skills present, plugin.json version verified.
- Cleanup: deleted 19 plugin-source rules + 2 lookup rules (`apple-health-no-rest-api`, `railway-domains`) + 5 v0.1 skill dirs from personal setup. (First attempt via Dave's other Claude Code session didn't actually execute the rm commands — agent commented but didn't invoke Bash. Re-ran from this session and verified.)
- Final personal `~/.claude/rules/` has 10 files; `~/.claude/skills/` is empty.

**Notes:**
- **Computer-use MCP injection overlap.** During conversion, this very session's system prompt included the computer-use MCP's auto-injected guidance (tier system, browser fallback, link safety, request-access flow). That guidance overlaps roughly half the original `computer-use-tiers.md` rule body. The skill body was trimmed to only include Dave-specific lessons not in that injection (Chrome vs Brave, login state, hybrid Computer-Use + Control Chrome pattern, Control Chrome reliability, iframe blind spots, Cowork agent fallbacks, form_input vs type on claude.ai). Net effect: smaller skill body, no duplicated guidance.
- **Cross-skill references.** `llm-metadata-extraction` originally referenced `llm-decision-scoping.md` by file path; updated to skill name. No other intra-plugin file-path refs found.
- **Forward-referenced links unchanged.** `ai-sdk-provider-defaults` still mentions `dave-tedder/open-brain-dashboard` — that repo exists. The earlier Session 1 note about a forward-ref to `github.com/dave-tedder/open-brain` (no -dashboard) was about the `retro` skill — content unchanged in v0.2.
- **iCloud-git-fragility** caveat continues to apply: this repo lives on iCloud. No corruption observed during this session's heavy file-write activity. README still warns cloners off iCloud.
- **Personal-rules at ~/.claude/rules/ inode-shared with iCloud** — verified ~/.claude is a symlink to `/Users/davetedder/Library/Mobile Documents/com~apple~CloudDocs/.claude/`, so editing either path edits the same files. Important for the upcoming deploy step: deleting from `~/.claude/rules/` deletes from the iCloud copy too.
- **Tracker change for Session 2:** added Task 7 (v0.2 refactor) to the todo list. Status flipped from COMPLETE back to IN PROGRESS.

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
