---
name: supabase-deploy-from-worktree
description: "Use when deploying a Supabase Edge Function via the Supabase CLI while working in a git worktree on a feature branch, or when debugging 'I changed the Edge Function but the behavior didn't update' — the deploy reads from the working directory, not git HEAD or main."
---

# Deploying Supabase Edge Functions From a Git Worktree

The Supabase CLI reads function source from the current working directory's `supabase/functions/<name>/index.ts` path. It does not read from the git index, HEAD, or main branch. This means:

- Deploying from a worktree on a feature branch deploys the **worktree's file state**, even if that commit is not yet merged to main.
- Deploying from the main repo directory deploys whatever is checked out there.
- An in-progress edit that hasn't been committed yet will still deploy if you save the file before running the deploy command.

## The gap this creates

After deploying from a worktree, live production is ahead of the main branch. If someone later re-deploys from main before the worktree is merged, they will regress the fix. This is the same failure mode as "I edited but didn't commit before deploying" but harder to spot because git status on main looks clean.

## Safer pattern

Pick one of these at session end:

1. **Merge the worktree branch into main before or immediately after deploy.** Keeps main and production in sync.
2. **Deploy from main after merging, not from the worktree.** Worktree is for safe editing; the final deploy commits the state change on main.
3. **If you must deploy from the worktree** (the usual case when a worktree was explicitly spawned for this task): push the branch immediately and flag the pending merge in the session log + next-task note, so the next session sees "live is ahead of main, merge X before touching Y."

## Detection

If a future session reports "I changed the Edge Function but the behavior didn't update," check:

1. Is there an open branch that has already shipped the fix? `git branch -a | grep claude/` surfaces unmerged claude-spawned branches.
2. Does the Edge Function on Supabase match what's in the main branch? Check via `supabase functions download` or the dashboard, compare to git.

## Real-world use

Open Brain Session 45 (2026-04-16) bumped the OpenRouter slug in `brain-digest/index.ts` and `compile-pages/index.ts`, then deployed from the worktree. The live functions picked up the fix, but the main `Open Brain/` working directory on disk still showed the old strings until the worktree merged. Flagged in that session's notes as a pending merge.
