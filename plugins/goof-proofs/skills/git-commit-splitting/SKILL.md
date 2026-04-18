---
name: git-commit-splitting
description: "Use when a single modified file contains hunks that belong to different logical commits (different sessions, different features, different bug fixes) and need to be split into separate commits — especially when `git add -p` isn't usable because no interactive stdin is available."
---

# Splitting Interleaved File Changes into Multiple Commits

When a single modified file contains hunks that belong to different logical commits (different sessions, different features, different bug fixes), the goal is to ship each hunk in its own commit without bundling unrelated work.

`git add -p` is the standard tool for this, but it requires interactive stdin, which is not usable from inside Claude Code's Bash tool. The reliable alternative is a save-reset-reapply cycle.

## The technique

1. Copy the working file to a scratch location outside the repo: `cp path/to/file.md /tmp/file_final.md`.
2. Reset the repo copy to HEAD: `git checkout HEAD -- path/to/file.md`.
3. For each logical commit group, use the `Edit` tool to apply only that group's changes against the HEAD baseline, then `git add` and `git commit`. The next group builds on the result of the previous group (the file is accumulating state in the working tree, just like a normal sequence of commits).
4. After the final group is committed, verify the reassembly is byte-exact: `diff path/to/file.md /tmp/file_final.md`. Empty output means every hunk landed in exactly one commit with no drift.

## Why this works

Each Edit operates against the current state of the file. If you land Session A's hunks first, then Session B's, then Session C's, the final file state is the same as if you'd made all the changes at once — the diff against the original working copy is zero. Any mismatch means a hunk was dropped, duplicated, or modified in transit.

The `diff` check is load-bearing. Without it, you're trusting that your mental model of the hunks matched the actual bytes, which is easy to get wrong with long files or similar adjacent changes.

## When to use it

- Uncommitted work that spans multiple sessions or topics and needs to be split for clean git history.
- Rebuilding a botched commit that bundled unrelated work (do the split on a fresh branch, cherry-pick the rest of history).
- Any case where `git add -p` would be the textbook answer but interactive input isn't available.

## When NOT to use it

If the file has only one logical change, just commit it. If two logical changes happen to share surrounding lines and can't be cleanly separated at the hunk level, document that in the commit message and bundle them rather than fighting the geometry.

Binary files can't be split this way — they're atomic. If a binary file needs splitting, that's a sign the single-file approach is wrong (probably needs a directory of per-topic assets instead).

## Real-world use

Open Brain Session 44 (2026-04-16) used this technique to split a single `CLAUDE.md` diff into four separate commits covering Session 39 auto-resolve conventions, Session 40 five-paths convention, Session 41 briefing notes, and a Routines-launch-day rewrite. The `diff` verification caught zero drift after the fourth Edit, confirming the four commits together rebuild the working state exactly.
