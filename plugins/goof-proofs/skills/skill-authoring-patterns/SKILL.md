---
name: skill-authoring-patterns
description: Use when authoring or editing a SKILL.md file, especially when writing YAML frontmatter descriptions, choosing trigger scope, genericizing personal guidance, or making a skill portable across agent tools.
---

# Skill Authoring Patterns

Use these conventions when writing `SKILL.md` files for Claude Code, Codex, Antigravity-style tools, or portable skill folders.

## Use "The User"

When a skill addresses a person, write "the user." Do not use a specific name or a placeholder like `{{USER_NAME}}`.

The host tool usually has global user preferences already. If the skill says "ask the user," the agent can still speak naturally in the actual conversation.

## Make The Description Load-Bearing

The YAML `description` is what the agent sees before deciding whether to load the full skill. A vague description means the skill may never fire, or it may fire constantly.

Good descriptions are narrow and situational:

- Good: "Use when writing or debugging code that passes an `anthropic/...` model slug to OpenRouter"
- Too broad: "Use when working with LLMs"
- Good: "Use when deploying a Supabase Edge Function from a git worktree"
- Too broad: "Use when deploying"

Use task situations, file paths, library names, command names, and error messages when they are natural signals.

## Describe The Moment Of Work

Most skills fire on a task, not a topic. Describe the moment when the agent should change behavior.

Examples:

- "Use when writing..."
- "Use when debugging..."
- "Use when deciding..."
- "Use when creating..."

## Split Mixed Skills

If one note covers two unrelated moments, split it into two skills. Tool-choice guidance and timeout recovery guidance may both involve the same tool, but they fire at different times.

## Preserve Useful Incidents

Real examples help agents recognize patterns. Keep safe incident summaries when they teach the shape of the problem. Remove secrets, private URLs, account IDs, embarrassing details, and business-only configuration.

## Keep Skills Portable

For cross-tool skills:

- say "the agent" instead of naming one tool everywhere
- mention both `~/.claude/skills/` and `~/.codex/skills/` when path differences matter
- keep tool-specific notes clearly labeled
- include scripts, examples, and templates inside the skill folder if the skill depends on them

## Real-World Use

The `goof-proofs` plugin uses these patterns to keep skill triggers narrow while making the bodies readable across users and tools. The goal is not to make every note tiny. The goal is to make the right note load at the right time.
