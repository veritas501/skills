---
name: plan-execute
description: Use when executing a plan-write multi-file plan directory (.plans/<MM-DD>-<feature_name>/ with 00-overview.md and tasks/), task-by-task. Use when plan execution drifts from the top-level design, when an executor ignores the agreed architecture, re-chooses tech, or breaks module boundaries documented in the plan.
---

# plan-execute

## Overview

Execute a layered multi-file plan (produced by `plan-write`) task-by-task, with the top-level design files as a **hard precondition** on every task.

**Core principle:** Drift happens when the executor works from task detail alone and forgets the top-level design. Prevent it by forcing a re-read of the relevant top-level files before each task, and a check against them after each task. The top-level design is not reference material — it is a gate.

## When to Use

- Executing a `.plans/<MM-DD>-<feature_name>/` directory with `00-overview.md` + `tasks/`
- Previous executions drifted: architecture ignored, tech re-chosen, boundaries broken
- Resuming a plan mid-way and needing to re-anchor to the design

## The Iron Precondition

```
NO TASK STARTS BEFORE READING ITS Depends-on FILES.
```

This applies to EVERY task, including ones that "look obvious". The whole point of the layered plan is that the design constrains the detail. Skip the read and you have a single-plan.md executor again.

**No exceptions:**
- Not for "I already read 00-overview earlier" — re-read the task's specific `Depends-on:` list each task.
- Not for "this task is trivial" — trivial tasks drift too.
- Not for "I remember the architecture" — memory drifts; the file is authoritative.

## Process

### Step 0: Load the contract (once)
Read `00-overview.md` fully. Hold its **Non-goals**, **Constraints**, **Acceptance criteria** as the session-wide guardrails. If any task later conflicts with these, STOP and ask — do not silently deviate.

### Step 1: Enumerate tasks
List `tasks/` in numeric order. Build a TodoWrite item per task.

### For each task:

1. **Read Depends-on** — open every file in the task's `Depends-on:` header. Re-read each task, even if read before.
2. **Restate constraints** — before writing code, restate (to yourself/the user) the specific boundaries this task must honor, from the "Constraints from top-level" section and the dependency files.
3. **Execute steps** — follow the task's steps exactly. Run verifications as written.
4. **Verify against top-level** — complete the task's "Verify against top-level" checklist. Confirm: output respects `20-architecture.md` boundaries; no tech introduced beyond `10-tech-stack.md`; interfaces match `40-interfaces.md`.
5. **Mark complete** only when both the task's own tests AND the top-level checks pass.

### Step N: Done
After all tasks: re-check every acceptance criterion in `00-overview.md` is met. Report any unmet ones — do not claim completion otherwise.

## Drift Detection — STOP conditions

Stop and ask the user when:

- A task step would violate a Non-goal or Constraint in `00-overview.md`.
- The cleanest implementation crosses a module boundary declared in `20-architecture.md`.
- A task needs a library/tool not in `10-tech-stack.md`.
- A task's `Depends-on:` file is missing or contradicts the task.
- An interface you must call doesn't match `40-interfaces.md`.

**Drift is not a shortcut.** When implementation reality conflicts with the design, the design is updated deliberately (with the user), not bypassed silently. Update the relevant top-level file first, then continue.

## Red Flags — you are about to drift

- "I'll just re-read the design later" → read it NOW, before the task.
- "This is simpler if I put it in module X instead" → that's a boundary change; STOP.
- "I'll add this small dep, it's fine" → not in tech-stack; STOP.
- "The task says X but I think Y is better" → raise it; don't silently substitute.
- "Acceptance criteria are basically met" → check each one explicitly.

**All of these mean: stop, re-read the top-level file, confirm or escalate.**

## Common Mistakes

| Mistake | Fix |
|---|---|
| Reading 00-overview once, never again | Re-read each task's `Depends-on:` per task. |
| Executing a task whose `Depends-on:` is empty | Plan defect — flag to user / route back to plan-write. |
| Silently choosing a different module than architecture says | STOP, update `20-architecture.md` with user, then proceed. |
| Claiming done without checking acceptance criteria | Re-verify every criterion in `00-overview.md` first. |
