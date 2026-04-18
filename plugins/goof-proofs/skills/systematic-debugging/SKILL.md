---
name: systematic-debugging
description: "Use when encountering any bug, error, or unexpected behavior before proposing fixes. Applies to runtime errors, build failures, deploys, database queries, API integrations, and automations across any stack."
---

# Systematic Debugging

## Iron Law

Find root cause before attempting any fix. No guessing. No random changes hoping something works.

---

## Phase 1: Root Cause Investigation

| Step | Action |
|------|--------|
| Read the error | Identify exact error message, file, line number, error code |
| Reproduce it | Confirm it happens consistently and under what conditions |
| Check recent changes | What changed just before this broke? Code, config, env vars, deploys |
| Trace data flow | Follow the data from input to the point of failure |

Do not move to Phase 2 until the failure point is isolated.

---

## Phase 2: Pattern Analysis

- Find a working example of the same operation (different endpoint, similar query, previous deploy)
- Compare working vs broken: what is structurally different?
- Check environment differences (local vs production, dev vs prod, one webhook vs another)
- Look at logs from just before the failure, not just at the moment of failure

---

## Phase 3: Hypothesis and Testing

- Form one specific hypothesis about the root cause
- Design the smallest possible test to confirm or disprove it
- Change one variable at a time
- If the hypothesis is wrong, revise it based on what the test revealed, then test again

Do not stack multiple changes in one attempt. Each change must be isolated and evaluated.

---

## Phase 4: Fix Implementation

- Fix the root cause, not the symptom
- Make the minimal change required
- Verify the fix actually resolved the original error
- Confirm nothing adjacent broke (check related endpoints, queries, or workflow steps)

---

## 3+ Fixes Failed: Stop

If three or more fixes have not resolved the issue, the debugging approach is wrong or the architecture has a deeper problem.

Stop and ask:
- Is the assumption about how this system works actually correct?
- Is this the right tool or approach for this problem?
- Is there a structural issue that point fixes cannot solve?

Reassess from scratch before attempting another fix.

---

## Red Flags (Bad Debugging Habits)

- Adding `console.log` everywhere without a clear hypothesis
- Trying fixes sequentially without understanding why each one should work
- Googling the error and copy-pasting solutions without reading them
- Changing multiple things at once so you can't tell what worked
- Assuming the framework, library, or platform is broken before ruling out your own code
- Fixing the symptom (e.g., suppressing an error) instead of the cause
- Not verifying the fix actually resolved the original failure
