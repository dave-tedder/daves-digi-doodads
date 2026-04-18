# LLM Decision Functions: Scope the Candidate Pool Before Asking

Any time you ask a cheap LLM (gpt-4o-mini, Haiku, etc.) to pick matches from a list — resolve open action items, pick relevant clients, match a thought to a project, classify a document against a taxonomy — the structural failure mode is always the same: if the candidate list contains items from unrelated contexts, the model will hallucinate matches no matter how conservative the prompt sounds.

"Be conservative" is not a constraint. It's a vibe. The model has already decided what a match looks like before it reads that sentence.

## The shape of the fix

1. **Scope the candidate pool at the DB layer before the LLM sees it.** Use concrete metadata (project, topic, tag, person, time window, status) to filter. One shared axis is the minimum bar for a candidate to even be considered. Unscoped inputs should short-circuit to empty rather than risk a spurious match.

2. **Inject per-candidate context into the list.** Don't send the model `1. description / 2. description / 3. description`. Send `1. [project=X, topics=Y/Z] description`. When the model sees mismatched context on an otherwise word-similar candidate, it has a concrete reason to skip it. This is your second line of defense if scoping is ever loosened.

3. **Require a quoted justification per match.** Change the output shape from `{"resolved":[1,3]}` to `{"resolved":[{"num":1,"reason":"short quote from the input"}]}`. Forcing the model to commit to a specific phrase in the input makes vague verb-collision matches ("fix", "update", "deploy", "check") mechanically impossible. The `reason` field doesn't have to be stored — extracting it is the point.

4. **Write hard rules in the prompt, not soft guidance.** Number them. Make them absolute. Example pattern: "1. The note must explicitly reference the item's subject. Generic verb overlap is NEVER enough. 2. The note must describe the action as DONE, SCHEDULED, or EXPLICITLY CANCELLED. 3. Cross-context matches are forbidden unless the note explicitly names the other context. 4. When unsure, return empty. Empty is the correct default."

## Real-world failure mode

Open Brain's `checkAutoResolve()` (Session 39, 2026-04-14) was pulling the top 50 globally-open action items and asking gpt-4o-mini which ones a new thought resolved. A Tedder Trainer bug got resolved by an Open Brain dashboard note. A Dollar Flight Club membership item got resolved by a McDonald's marketing email ("free" keyword collision). A UptimeRobot user survey got resolved by a session log about cutting a stat counter. The LLM had no way to know these were different projects because the project was never in the prompt.

Fix was structural, not prompt-tweaking alone: batched source metadata in via `.in("id", sourceIds)`, filtered candidates by project OR topic OR person match with the new thought, then sent a stricter prompt with per-candidate context and a required reason quote. After the fix, the same system correctly resolved one genuinely-related item on a test capture and skipped everything else including a canary item that shared the same project axis.

## Detection

If you're debugging false positives in an LLM picker:
1. Count how many candidates the LLM sees per call. Anything over ~10 with no scoping is suspect.
2. Check whether the candidates are structurally comparable (same project, same taxonomy, same type) or a random global pool.
3. Read the prompt. If the constraint text is "be conservative" or "only when clear", that's the bug.
4. Look at the output shape. If it's just a list of indices with no justification, the model has no accountability on individual picks.
