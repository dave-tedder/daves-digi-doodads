---
name: pre-build
description: "Use before any non-trivial implementation task: building something new, designing an automation, integrating a new tool, or making a significant change to an existing workflow."
---

# Pre-Build Skill

## Purpose

Stop and think before building. The goal is to land on the right approach before writing a single line of code or touching any live system.

## When to Use

Trigger this skill when the task involves:
- A new automation or workflow (CMS/content system, database, deployment, LLM integration)
- Integrating two or more systems together
- Changing how an existing process works
- Building something that will be hard to undo

Do not trigger for simple, clearly-scoped tasks (fix a typo, add one field to a form, run a command).

## Process

**Step 1: Understand the intent.**
Before asking anything, restate what the user is trying to accomplish in one or two sentences. Confirm understanding. If it's off, correct it before proceeding.

**Step 2: Ask one question at a time.**
Identify the single most important unknown and ask it. Wait for the answer. Do not front-load a list of questions.

**Step 3: Propose 2-3 approaches.**
Once there is enough context, present 2-3 concrete options. For each, state:
- What it does
- What it costs (time, complexity, money if relevant)
- The tradeoff

Lead with the recommendation and say why. Be direct.

**Step 4: Get approval. Do not build until approved.**
This is a hard gate. Do not begin implementation until the user confirms the approach. If they modify the approach, confirm the revised plan before proceeding.

**Step 5: Determine version control strategy.**
Ask the user: "Should this project use git for version control?" If yes:
- Initialize the repo (or confirm it already exists).
- This activates the Save Points protocol: one commit per completed task, no exceptions. This is the rollback safety net. If work is committed to git, every task gets its own commit before the next task starts.
- Add this to the PROJECT-TRACKER.md under Prohibited Actions: "Never batch multiple tasks into a single commit. Each task gets its own commit as a rollback point."

If no (CMS-only workflows, no-code platforms, etc.): the tracker progress log is the only recovery record. Note this in the tracker.

**Step 6: Set up the project file tree, including a project-level CLAUDE.md.**
Once approved, create the initial file structure for the project. This includes a `CLAUDE.md` in the project root. The CLAUDE.md should contain:
- One-paragraph project description (what it is and does)
- Tech stack and key dependencies
- Project structure (directory tree of important paths)
- Key files with brief descriptions of what each one does
- Database schema or data model (if applicable)
- How to run, build, test, and deploy
- Conventions and patterns specific to this codebase
- Current status

Keep it concise and factual. No secrets or credentials in this file (reference where they're stored instead). This file loads automatically in every Claude Code session, so it should give a fresh agent everything it needs to start working immediately.

Also create the `PROJECT-TRACKER.md` per the tracking protocol.

## Design Principles

**YAGNI.** Only design for what is needed now. Do not add flexibility, abstraction, or future-proofing unless the user asks for it.

**Prefer existing tools.** Reach for what's already in the user's stack before suggesting anything new. New tools add maintenance cost, new accounts, new failure modes.

**Smallest working solution first.** If the simplest approach covers 90% of the need, recommend it. Complexity is a cost.
