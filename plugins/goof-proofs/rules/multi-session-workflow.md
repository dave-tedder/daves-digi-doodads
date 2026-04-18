# Multi-Session Workflow Protocol

## When to Break Work Into Multiple Sessions

Evaluate every project and prompt-crafting request for complexity. Split into multiple chat sessions when:

- The total work would exceed what can be reliably completed in a single context window.
- The project has natural phases where one phase must be verified before the next begins.
- the user is asking for help writing instructions or prompts for a multi-step project.
- The work involves multiple independent systems that each need focused attention.

When in doubt, smaller sessions with clean handoffs are better than one massive session that loses coherence.

---

## How to Structure Multi-Session Work

### Phase Planning
At the start of a multi-phase project:

1. Map out all phases with clear boundaries.
2. Define what "done" looks like for each phase.
3. Identify dependencies (what must be finished before the next phase can start).
4. Write this plan into the PROJECT-TRACKER.md.

### Phase Handoff Format
At the end of each phase, provide the user with:

1. A summary of what was completed in this phase.
2. Confirmation that the phase is verified and working.
3. Clear instructions to start a new chat for the next phase.
4. A ready-to-paste prompt for the next agent, formatted in a code block.

The handoff prompt must include:
- Project name and location of the PROJECT-TRACKER.md.
- Which phase was just completed.
- Which phase is next.
- Any context the next agent needs that isn't in the tracker or session log.
- Instruction to read the PROJECT-TRACKER.md first, then the latest SESSION-LOG.md entry.

Example format:

```
---
NEXT PHASE PROMPT (copy and paste this into a new chat):
---

I'm working on [project name] located at [path]. Phase [N] ([phase name])
is complete. Please read the PROJECT-TRACKER.md first, then the latest
entry in SESSION-LOG.md, and continue with Phase [N+1]: [phase name].

The tracker has the todo list and status. The session log has detailed
history. [Any additional context the next agent needs].
```

---

## Session Start and Close

### Starting a Session
When beginning work on a project (new or resumed):

1. Read PROJECT-TRACKER.md for status and next task.
2. Read the latest SESSION-LOG.md entry for context on what just happened.
3. Run `git status` and handle any uncommitted work before touching new files (see "Pre-work git state check" below).
4. Create a new session entry in SESSION-LOG.md with the next session number and today's date.
5. Begin work.

### Pre-work git state check

Before starting any task work, evaluate the output of `git status` (or the "Status" block the chat environment prints at session start).

- **Clean working tree:** proceed with the session.
- **Uncommitted/untracked files from a previous session** (the typical case when the prior session logged a planning pass or flagged carry-over under its **Notes**):
  1. Ask the user whether to (a) commit as a standalone "Session N carry-over" commit before starting new task work, (b) fold into the first task commit of the current session, or (c) leave as-is for now. Default preference: (a), because it keeps each of the current session's task commits as a clean rollback point.
  2. Do NOT silently fold prior-session work into the current session's first task commit. That blends attribution and muddies the rollback story.
- **Uncommitted files with no obvious origin:** investigate before touching. Never run `git restore`, `git checkout -- <file>`, or deletion as a way to "clean up" unknown work. See `~/.claude/rules/icloud-git-fragility.md` for the concurrent-session iCloud sync hazard.

The goal is for each session's contribution to git history to stand on its own: the commits in a session map 1:1 (or close to it) to the tasks that session executed. Leftover work from a prior session gets its own commit or an explicit "folded into Task N" decision from the user, never a silent merge.

### Closing a Session
Before ending a session:

1. Finalize the current session entry in SESSION-LOG.md with all fields filled in.
2. Update the Session Index at the top of SESSION-LOG.md.
3. Update the "Last Completed" section in PROJECT-TRACKER.md.
4. Capture an Open Brain thought summarizing the session (see tracking-and-verification.md).
5. If the next phase requires a new chat, provide the handoff prompt.

---

## When Crafting Prompts or Instructions

When the user asks for help writing prompts or instructions for a project:

1. Consider the full scope of what he's trying to accomplish.
2. Factor in his available tech stacks (see tech-stacks.md).
3. If the work would benefit from multiple sessions, structure the instructions as phased prompts rather than one monolithic prompt.
4. Each phase prompt should be self-contained enough that a fresh agent can execute it with only the prompt, PROJECT-TRACKER.md, and SESSION-LOG.md for context.
5. Include in each prompt: what tools/platforms are involved, what the expected outcome is, and where to find any previous work.

---

## Context Recovery

If the user says something like "continue where we left off" or "pick up from the last session":

1. Ask which project (if not obvious).
2. Read the PROJECT-TRACKER.md.
3. Read the latest SESSION-LOG.md entry.
4. Summarize the current state back to the user before doing anything.
5. Confirm the next task before proceeding.
