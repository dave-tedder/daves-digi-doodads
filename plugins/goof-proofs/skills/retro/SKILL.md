---
name: retro
description: "End-of-session retrospective. Captures what worked, what broke, decisions made, and feeds learnings back into rules/memory so the next agent benefits. Run this before closing out any non-trivial session."
---

# Session Retrospective

## Purpose

Structured reflection at the end of a work session. Captures outcomes, surfaces patterns, and routes learnings to the right persistence layer (rules, memory, CLAUDE.md, or your own memory system) so future agents start smarter.

## When to Use

Call `/retro` when wrapping up a session where any of the following happened:
- Bugs were found and fixed
- Workarounds or non-obvious solutions were discovered
- Architectural or tooling decisions were made
- Something broke unexpectedly and you had to change approach
- A multi-task sequence was completed on a tracked project

Skip it for quick, single-task sessions (toggled a setting, answered a question, made one small edit).

## Process

**Step 1: Gather context.**
Read these sources (skip any that don't exist for the current project):
- `PROJECT-TRACKER.md` in the project root (progress log entries from this session)
- Recent entries in your memory system, if you use one (last 24 hours, filtered to relevant topics — e.g., [Open Brain](https://github.com/dave-tedder/open-brain) or similar)
- Git log for commits made this session (if git is active)
- Any files created or modified during the session

Do not ask the user to summarize the session. The point is to pull this from the record automatically.

**Step 2: Generate the retrospective.**
Present a concise summary organized into these sections:

### What got done
Bullet list of completed tasks. Keep it factual, one line each.

### What broke or surprised us
Anything that didn't go as expected. Root causes, not just symptoms. If nothing broke, say so and move on.

### Decisions made
Architectural choices, tool selections, approach changes, scope adjustments. Include the reasoning.

### Learnings worth keeping
Things a future agent would benefit from knowing. This is the most important section. Be specific and actionable. Examples:
- "Video avatar model outputs 480x832 at 25fps, acceptable for 2D art"
- "Notion Calendar URLs encode a different ID than Google Calendar, need title-based fallback"
- "Railway OOM at default memory when Promise.all downloads 9+ assets simultaneously"

### Deferred or unfinished
Anything that got pushed to next session, flagged but not addressed, or left in a partially-complete state.

**Step 3: Route the learnings.**
For each item in "Learnings worth keeping," recommend where it should go:

| Destination | When to use it |
|---|---|
| `~/.claude/rules/[topic].md` | Cross-project principle (applies to more than one project) |
| Project CLAUDE.md | Project-specific convention or gotcha |
| Project memory file | Context a future agent needs for this specific project |
| Your memory system (e.g., Open Brain) | Observation worth capturing for semantic search later |
| Nowhere | Already captured elsewhere, or too ephemeral to store |

Present the routing as a table. Do not write anything yet.

**Step 4: Get approval, then persist.**
Show the user the routing table and ask: "Want me to save all of these, or adjust any?" Once confirmed, write each learning to its destination. For rules and memory files, follow the existing format conventions (frontmatter for memory, plain markdown for rules). Update MEMORY.md indexes if new memory files are created.
