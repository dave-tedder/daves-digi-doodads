# Project Tracking & Verification Protocol

Every project uses two tracking files: `PROJECT-TRACKER.md` (what needs to happen) and `SESSION-LOG.md` (what already happened). Both are required. No exceptions.

## Never Overwrite Tracking Files

Before writing to PROJECT-TRACKER.md or SESSION-LOG.md, you MUST Read the file first. These files accumulate session history across the entire project lifecycle. If a file search returns no results, verify with `git ls-files` or `ls` before assuming the file doesn't exist. Never use Bash heredocs (`cat > file << EOF`) to write these files, as it bypasses the Write tool's read-first safety check. Always use the Edit tool for updates.

## File Templates

### PROJECT-TRACKER.md

```
# [Project Name] - Tracker
## Status: IN PROGRESS
Started: [date] | Last Updated: [date]

## Todo List
- [ ] Task 1 - description [Risk: low/medium/high]

## Prohibited Actions
- Never commit secrets or API keys
- [Add project-specific guardrails]

## Last Completed
**Session:** [number] ([date])
**What was done:** [1-2 sentence summary]
**Next task:** [what comes next]
```

### SESSION-LOG.md

```
# [Project Name] - Session Log
## Session Index
<!-- One line per session. Newest at top. Format: Session N (date) - [summary] -->

## Session 1 - [date]
**Task(s) completed:** [task names]
**What was done:** [detailed changes, decisions, gotchas]
**Files modified:** [list]
**Verification:** [what was checked]
**Commit(s):** [hash(es) if git]
**Notes:** [anything next agent needs]
```

## During Work

After completing EACH task:

1. **PROJECT-TRACKER.md:** Mark task complete, update "Last Completed" (overwrite previous), update date. Add newly discovered sub-tasks.
2. **SESSION-LOG.md:** Add details to current session entry, update Session Index.
3. **Open Brain:** Capture a thought: `[Project Name] Session [N] - [task]: [summary]`. Tag with project name and "session-log". Skip if Open Brain MCP unavailable.

## Verification

Before marking any task complete:
- The change works as intended (test it, don't assume)
- Nothing previously working is broken
- For code: runs without errors, existing tests pass
- Session log entry has enough detail for a fresh agent to continue

**"File parses" is not verification.** When fixing a CSS, runtime, or behavioral bug, confirming that Vite/tsc/the build succeeds only proves the edit is syntactically valid. It does not prove the bug is fixed. Functionally test the fix itself: for CSS bugs, inject the rule in the browser preview (`preview_eval`) and read `getComputedStyle` on synthetic DOM; for runtime bugs, exercise the failing code path; for API bugs, hit the endpoint. If you ship a "fix" based on a diagnosis without testing the diagnosis, expect to ship a second fix minutes later.

**"Unit tests pass" is not verification either, for anything that aggregates, categorizes, or transforms real-world data.** A unit-tested `parseQuantity("1/2 cup")` and `categorizeIngredient("beef broth")` will still ship three wrong answers if you never run the pipeline end-to-end against a real row from the database. Tedder Trainer Session 43 (2026-04-14): wrote the grocery list builder, all 8 unit tests passed, then ran it against the 7 real recipes from the active meal plan and immediately found three bugs that the unit tests couldn't see:
- Fraction regex ordering — `"1/2 cup"` parsed as `"1"` with a stray `"/2 cup"` unit. The unit test for `"1/2 cup"` happened to pass because the test was written against the same flawed implementation.
- `"beef broth"` categorized as Meat because the `beef` regex fired before the `broth` check. No unit test specifically asserted `"beef broth" → Pantry`.
- `"diced tomatoes"` fell through to Other because `\btomato\b` doesn't match the plural. The unit tests used singular `"tomato"`.

Integration test against production data is a separate, mandatory step. Pull the real rows, run the helper, and assert on the end-to-end output. Unit tests prove the pieces do what you wrote them to do; integration tests prove you wrote the right pieces.

## Git Save Points

One commit per completed task. No batching. Stage only files from that task, commit as `Task N: [description]`, confirm clean `git status` before starting next task.

Non-git projects (WordPress MCP, Notion): skip commits, session log is the recovery record.

Medium/high risk tasks: confirm approach with the user before executing.

## Resuming a Project

1. Read `PROJECT-TRACKER.md` first (todo list, last completed, next task).
2. If more context needed, read latest `SESSION-LOG.md` entry.
3. Check for partially started work from previous session.
4. If something looks incomplete, investigate before continuing.

## Maintenance

**Tracker:** Keep under 200 lines. When all tasks in a phase are done, collapse to `### Phase N: [Name] - COMPLETE` and remove individual items. Check size at session start.

**Session log:** Keep last 15 sessions in active file. Rotate older sessions to `SESSION-LOG-ARCHIVE.md`. The Session Index always lists all sessions (mark archived ones).

**Session log accuracy:** At session close, verify the current entry has final numbers, all changes actually made, and resolved items updated. Mid-session entries go stale.

## Migrating Old Projects

If a project has progress log entries inside PROJECT-TRACKER.md (old format): create SESSION-LOG.md, build Session Index from existing entries, use new format going forward. Migrate incrementally, not all at once.

## Project Completion

1. Set tracker Status to COMPLETE.
2. Add final session log entry (outcome, follow-ups, known issues).
3. Capture Open Brain thought: `[Project Name] COMPLETE: [summary]`

## Flagging Pre-Existing Uncommitted Work at Session End

If you reach session close and there is uncommitted or untracked work in the repo that you did NOT create this session (it was already there when you started and was deliberately left out of your own commits), call it out explicitly in your SESSION-LOG.md entry under **Notes**. Name the files, say they pre-dated the session, and recommend what the next session should do (commit as-is, split by topic, discard, etc.).

This is a self-documenting handoff. It survives context loss between chats without needing an external tracker, and it gives the next agent a concrete starting point — "read the prior session's Notes" is a much smaller context hand-off than "inspect the whole working tree and figure out where everything came from."

Example phrasing that works (from Open Brain Session 43):

> This session also touched CLAUDE.md, brain-digest, ingest-thought — those modifications pre-dated this session (visible in git status at start) and should be reviewed before committing, or split into a separate commit.

When the next session picks it up, they have the file list, the origin context ("pre-dated this session"), and a suggested approach ("split into a separate commit"). The cleanup in Open Brain Session 44 took one pass because of exactly this note — without it, reconstructing the attribution for a week-old diff would have been considerably more expensive.

**Corollary: don't write retroactive session log entries for work you didn't do.** If triaging uncovers that some past commit has no corresponding SESSION-LOG entry, flag the gap in the current session's Notes — name the commit SHA and the date — rather than fabricating an entry for a session you weren't present for. The commit message is already the record; the session log just indexes it.
