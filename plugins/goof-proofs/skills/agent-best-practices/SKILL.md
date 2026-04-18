---
name: agent-best-practices
description: "Use when planning any multi-step task involving agents, subagents, or work that may span multiple sessions. Covers parallel vs sequential agent use, context management, subagent task design, multi-session handoffs, and verification discipline."
---

# Agent Best Practices

## Parallel vs Sequential Agents

**Run agents in parallel when:**
- Tasks are fully independent (no shared files, no shared state)
- Research and writing can happen simultaneously
- Multiple unrelated file operations are needed
- Order does not matter and neither task's output feeds the other

**Run agents sequentially when:**
- Task B needs Task A's output to proceed
- Tasks touch the same files or database records
- You need to verify Task A worked before starting Task B
- Shared state would cause race conditions or conflicts

When in doubt, sequential is safer. Parallel only when independence is certain.

---

## Context Management

- Monitor `/context` during long sessions. Warn the user when free space drops below 30%.
- Long conversations degrade coherence. Start a new session with a handoff prompt rather than pushing through a bloated context.
- Subagents have their own isolated context. They do not inherit the parent conversation. Everything they need must be in the prompt you give them.

---

## Subagent Task Design

Every subagent prompt must be self-contained. Include:

1. What was done before this task (relevant prior state)
2. What the task is, precisely
3. Where the relevant files are (absolute paths)
4. What "done" looks like (acceptance criteria)
5. What to do if something looks wrong

Do not rely on the subagent inferring context from scratch. Assume zero prior knowledge. If the subagent needs to read a file first, say so explicitly.

---

## Multi-Session Handoffs

Before ending any session on a multi-phase project:

1. Create or update `PROJECT-TRACKER.md` in the project root.
2. Log: what was completed, files modified, verification status, what comes next.
3. Provide a ready-to-paste prompt for the next session.

Handoff prompt format:

```
I'm working on [project name] at [path]. Phase [N] ([name]) is complete.
Read PROJECT-TRACKER.md first, then continue with Phase [N+1]: [name].
[Any critical context not captured in the tracker.]
```

---

## Verification Discipline

Do not mark a task complete until:

- The output was actually tested, not just written
- Previously working functionality is confirmed still working
- For code: it runs without errors
- For content/config changes: the result is visually or functionally confirmed

Update the tracker after every task. The progress log is the safety net for the next agent. Write entries as if a fresh agent with no context will read them, because one will.
