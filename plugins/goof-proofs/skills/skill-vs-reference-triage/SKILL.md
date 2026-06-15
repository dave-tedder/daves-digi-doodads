---
name: skill-vs-reference-triage
description: Use when deciding whether reusable agent guidance should become a triggered skill, a reference markdown file, or a split between both.
---

# Skill Vs Reference Triage

Use this when deciding how to package reusable agent guidance.

## The Test

Make it a skill if the guidance changes the agent's behavior at the moment of work. It tells the agent how to do something, and the user benefits from the agent applying it without remembering to ask.

Make it a reference file if the guidance documents a fact, constraint, or one-time setup procedure. The user benefits from being able to find it when needed, but automatic loading adds little.

## Good Skill Candidates

- Writing Supabase queries with safer insert and error handling patterns
- Authoring a Next.js server component without passing event handlers through a server boundary
- Asking an LLM to pick matches from a list after narrowing the candidate pool
- Debugging a specific recurring failure mode with a known process

These fire while code, prompts, or tool choices are being made.

## Good Reference Candidates

- A one-time setup checklist
- A platform fact, such as an API that does not exist
- A broad project workflow convention
- A troubleshooting note the user will explicitly ask for when the problem appears

These are useful to read, but usually too broad or too occasional to trigger automatically.

## Split Mixed Guidance

If one note covers two distinct moments, split it.

Example: tool-selection guidance belongs in one skill, while timeout recovery for a specific tool belongs in another. Those happen at different points in the work, so one combined trigger would load extra content half the time.

## Trigger Quality

A narrow skill with a longer body is better than a short skill with a broad trigger. The description is the always-visible part. The body loads only when the trigger matches.

Use file paths, library names, command names, error messages, and concrete task situations in the description when they help the agent recognize the right moment.

## Default

When uncertain, prefer a skill with a narrow trigger. It is cheaper than putting broad reference material into always-loaded global instructions.

## Real-World Use

The `goof-proofs` v0.2 refactor used this test to move recurring behavioral guidance into triggered skills while leaving broad workflow docs as cherry-pickable reference rules. That kept the full bundle useful without forcing every note into every session.
