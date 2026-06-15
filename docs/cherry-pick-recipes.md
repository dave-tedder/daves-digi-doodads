# Cherry-Pick Recipes

Use these recipes when you do not want the full bundle.

## Full Bundle

Best for Claude Code users who want all 29 skills available.

```bash
/plugin marketplace add dave-tedder/daves-digi-doodads
/plugin install goof-proofs@daves-digi-doodads
```

This keeps skill bodies lazy-loaded. The reference rules still need to be copied or read manually if you want them active.

## Project Tracking Only

Best for any agent tool where continuity matters across sessions.

Copy into your project root:

```text
templates/project/AGENTS.md
templates/project/PROJECT-TRACKER.md
templates/project/SESSION-LOG.md
```

Then fill in real project commands, guardrails, and the first task.

## Debugging Discipline Only

Best when agents jump to fixes too early.

Copy:

```text
plugins/goof-proofs/skills/systematic-debugging/SKILL.md
```

## LLM Prompting Only

Best for extraction, matching, and model-provider gotchas.

Copy these skills:

```text
plugins/goof-proofs/skills/llm-decision-scoping/SKILL.md
plugins/goof-proofs/skills/llm-metadata-extraction/SKILL.md
plugins/goof-proofs/skills/openrouter-slug-format/SKILL.md
plugins/goof-proofs/skills/ai-sdk-provider-defaults/SKILL.md
```

## Skill And Plugin Authoring

Best when you are building your own reusable skills, rules, or plugin docs.

Copy these skills:

```text
plugins/goof-proofs/skills/gh-username-verification/SKILL.md
plugins/goof-proofs/skills/skill-authoring-patterns/SKILL.md
plugins/goof-proofs/skills/skill-vs-reference-triage/SKILL.md
```

## Supabase And Next.js Helpers

Best for projects using Supabase, Next.js App Router, or both.

Copy these skills:

```text
plugins/goof-proofs/skills/supabase-query-patterns/SKILL.md
plugins/goof-proofs/skills/supabase-deploy-from-worktree/SKILL.md
plugins/goof-proofs/skills/nextjs-middleware-auth/SKILL.md
plugins/goof-proofs/skills/nextjs-server-component-events/SKILL.md
plugins/goof-proofs/skills/nextjs-server-component-verification/SKILL.md
```

Optional reference:

```text
plugins/goof-proofs/rules/security-baseline.md
```

Copy the reference only if you want those checks loaded into your project instructions.

## Brain Bank Companion Habits

Best when you use a memory system and want clean session closeouts.

Copy:

```text
templates/project/PROJECT-TRACKER.md
templates/project/SESSION-LOG.md
plugins/goof-proofs/skills/retro/SKILL.md
plugins/goof-proofs/rules/tracking-and-verification.md
plugins/goof-proofs/rules/multi-session-workflow.md
```

Adapt the closeout step to your actual memory tool. Do not copy private URLs, keys, database IDs, or account names into a public repo.
