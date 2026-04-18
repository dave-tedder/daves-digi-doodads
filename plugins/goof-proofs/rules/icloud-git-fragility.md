# iCloud Drive + Git Fragility

Git repositories stored on iCloud Drive are prone to corruption. iCloud's sync mechanism can delete or fail to sync individual objects in `.git/objects/`, leave stale `.lock` files, and abandon in-progress operations (rebase-apply, am).

## Symptoms

- `fatal: bad object HEAD`
- `git cat-file: could not get object info`
- Stale `.lock` files or `rebase-apply`/`rebase-merge` directories with no active operation
- Missing pack files or loose objects

## Recovery

1. Check for abandoned operations first: `git am --abort` or `git rebase --abort`, remove stale `.lock` files.
2. If objects are missing, the fastest fix is re-cloning from the remote: clone to `/tmp`, copy the `.git` directory back, verify with `git log` and `git status`.
3. All working files are fine (iCloud syncs those reliably). It's only `.git` internals that break.

## Prevention

- Push frequently. The remote is the source of truth, not the local `.git`.
- After any interrupted git operation (Ctrl-C, crash, iCloud conflict), check `git status` before doing more work.
- Consider using `git gc` periodically to pack loose objects (fewer files = fewer sync issues).

## Concurrent Session Hazard

If two Claude sessions edit the same tracked files, iCloud sync can cause one session's changes to appear in another session's commit. The file contents are correct, but `git status` shows clean and the changes are already in HEAD from the other session's commit. Always verify with `git show HEAD -- <file>` (not just `git status`) when working across concurrent sessions on the same repo.

## Sibling Worktree Grep Noise

Worktrees live at `.claude/worktrees/<name>/` inside the repo, each checked out to its own branch. A recursive grep across `Projects/Apps/` (or any directory containing multiple repos with worktrees) walks every worktree and shows stale matches from branches that haven't merged yet. This is expected behavior, not an incomplete fix.

When doing a verification sweep after a cross-repo change:

- Filter out sibling worktrees: `grep -v '\.claude/worktrees/'` or pass `--glob '!**/.claude/worktrees/**'` to ripgrep.
- Or, scope the sweep to only source paths that matter: `grep -rn ... server/ supabase/ src/ docs/` instead of the whole project tree.
- The current-working worktree (the one the active session is editing) is not noise — only *other* worktrees count as sibling noise.

Real-world case: Open Brain Session 45's post-fix sweep for `anthropic/claude-sonnet-4` returned 17 hits, 13 of them from three sibling worktrees on unrelated branches. Filtering those out left the 4 hits that were already fixed in the active worktree (code + prose in CLAUDE.md + the dashboard-redesign prompt) plus 1 audit doc self-reference, all expected.
