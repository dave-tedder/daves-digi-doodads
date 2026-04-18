---
name: llm-metadata-extraction
description: "Use when writing or debugging an LLM prompt that extracts structured metadata (action items, todos, tags, dates, project fields, categories, follow-ups) from narrative text, or when co-generating creative content alongside a derived structured list (meal plan + grocery list, video script + shot list, etc.)."
---

# LLM Metadata Extraction Prompts

Lessons from running gpt-4o-mini as a structured-metadata extractor in production (Open Brain's `extractMetadata()`). Applies to any prompt that asks an LLM to parse narrative text into JSON fields like action items, project tags, topics, people, due dates, priority, etc.

## Past-tense work gets silently converted to imperatives

gpt-4o-mini will happily read "I updated the ratio to 3px/1px, replaced the background, shipped the fix" and emit `action_items: ["Update the ratio to 3px/1px", "Replace the background", "Ship the fix"]`. It defaults to generous — any verb phrase becomes a to-do unless you explicitly forbid it.

"array of implied to-dos (empty if none)" is not enough guidance. You need explicit negative examples and a default-to-empty baseline for the common case.

**Pattern for action-item / todo extraction:**

```
"action_items": array of FUTURE to-dos the user still needs to do. STRICT RULES:
  * Only include items phrased as future work: "need to X", "should X",
    "TODO: X", an imperative about something not yet done, or an explicit
    open question.
  * NEVER include work described in past tense. If the note says "I updated X",
    "fixed Y", "shipped Z", those are DONE and must be excluded.
  * NEVER restate completed work as an imperative. Do not turn "updated the
    ratio to 3px" into "Update the ratio to 3px".
  * Session logs, changelogs, retrospectives, and "here's what I just did"
    summaries almost always have an empty action_items array. Default to []
    when in doubt.
  * A commitment to do something later ("I'll test this tomorrow") IS an
    action item. A description of something already tested is NOT.
```

The negative examples are doing most of the work. The model needs to see the shape of what to exclude, not just hear the rule.

Applies to any extraction field that pulls imperatives from narrative text: "action_items", "todos", "follow_ups", "next_steps", "open_questions".

## Hallucinated tag/category/project fields on unrelated inputs

gpt-4o-mini will happily tag a Duke Energy utility bill as `project="Tedder Trainer"` if a recent context window contained Tedder Trainer notes. It will tag a TaxAct marketing email the same way. The model leans hard on topical similarity and recent context, not explicit references, when filling free-form category fields.

This matters when the downstream logic depends on the extracted category being accurate — scoped auto-resolve, project-based routing, wiki compilation, digest briefings. Garbage-in-garbage-out at the scoping layer.

**Pattern for free-form category fields:**

```
"project": the project or system this note is ABOUT. Only fill this when
  the note explicitly references a known project by name or is clearly
  session-log content for one. If the note is a marketing email, utility
  bill, tax reminder, or random inbox item with no project context, return
  null. Do NOT guess a project from topical similarity.
```

Explicit "do NOT guess from topical similarity" instruction is load-bearing. Without it, the model's default behavior is to guess.

If the hallucination keeps happening despite the prompt tightening, the next layer is a hard-coded pre-pass: for captures from known-noisy sources (email bridges, marketing scrapers), force the field to null unless the text matches a known project name. Don't bother until the prompt fix proves insufficient in production — explicit regex rules are higher maintenance than prompt rules.

## Require justification quotes for claimed matches

If the LLM is making a pick or a match decision (not just extracting fields), change the output shape from `{"matches":[1,3]}` to `{"matches":[{"num":1,"reason":"short quote from the input"}]}`. Forcing the model to produce a concrete phrase from the input text makes vague pattern-matching mechanically harder. The model has to commit.

The `reason` field doesn't have to be stored anywhere. The act of generating it is the self-check. But if you do log it (console.log is fine), you get free audit trails on every decision the LLM made — invaluable for post-mortem when a false positive slips through.

Your parser should accept both the object form and the legacy number form so the LLM can downgrade gracefully:

```ts
const arr = Array.isArray(parsed.matches) ? parsed.matches : [];
const nums = arr
  .map((entry: unknown) => {
    if (typeof entry === "number") return entry;
    if (entry && typeof entry === "object" && "num" in (entry as Record<string, unknown>)) {
      const n = (entry as Record<string, unknown>).num;
      return typeof n === "number" ? n : NaN;
    }
    return NaN;
  })
  .filter((n: number) => Number.isFinite(n));
```

## Don't co-generate creative content + structured enumeration in one LLM call

If an LLM prompt asks for both a creative artifact AND a derived structured list (e.g. "generate a meal plan AND the grocery list for it," "write a video script AND the shot list," "design a workout program AND the equipment list"), the structured enumeration is unreliable even when the creative output is good. The model is great at the vibes pass and sloppy at the counting pass, and it's already "done" by the time it gets to the list.

Failure modes observed in production:
- **Dropped items.** The meal plan called for broccoli, soy sauce, sesame oil, ginger, brown rice, and chicken thighs across multiple recipes. None of them made the grocery list. Claude got distracted by the variety of the plan and moved on.
- **Wrong quantities.** Recipe called for 1.5 lb salmon. Grocery list said 2 lb. Code can `sum(recipe.ingredients)` exactly. The model guesses.
- **Hallucinated items with no source.** The plan used a sirloin recipe for Sunday. The grocery list emitted "ground beef 2 lb" — no recipe in the plan used ground beef at all. This is worse than dropped items because a verification step that only checks "every recipe ingredient is on the list" won't catch an item that was invented. Bidirectional verification is required: every list item must trace back to a specific source in the data.

**Pattern: split the call, or do the aggregation in code.**

The LLM generates the creative output (meal plan, script, program). Code then:
1. Reads the structured data referenced by the creative output (e.g. `recipes.ingredients` for each recipe_id in the plan).
2. Aggregates deterministically (sum quantities, normalize names, categorize).
3. Emits the final list.

This matches the `llm-decision-scoping` skill's principle: let the LLM do the parts it's good at (vibes, language, pattern-matching), let code do the parts it's good at (counting, sums, enumeration). Tedder Trainer Session 43 (2026-04-14) fixed the meal plan generator by removing the `grocery_list` from the save path — Claude still emits one in its JSON output, but it's thrown away and replaced by `buildGroceryListFromMeals(plan.meals)`. All 11 previously-dropped ingredients now appear on every plan.

If splitting isn't feasible (e.g. a single-shot tool response that must contain both), at minimum run a code-side bidirectional check BEFORE persisting:
- Every source item appears in the list (no drops).
- Every list item appears in at least one source (no hallucinations).
- Sum of list quantities equals sum of source quantities per ingredient.

Log a warning and regenerate or reject if any check fails. Don't just log and keep going.

## What this replaces

Before Session 39 (2026-04-14), Open Brain's `extractMetadata` prompt was a single "array of implied to-dos (empty if none)" line and the `project` field said "e.g. Open Brain, UFO Pipeline… (null if general/unclear)". Both produced daily false positives in production. The rewrite with negative examples and hard rules cleaned them up in one pass.
