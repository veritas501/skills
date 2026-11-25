---
name: plan-execute
description: Execute or resume a plan-write plan directory task by task, recording durable results.
---

# plan-execute

Coordinate one plan directory from validation through final acceptance.

Three sources have different authority, and none stands in for another:

- The plan (`00`–`50` design files and `tasks/`) records the agreed change and its boundaries. It is frozen once execution starts.
- The repository (code, tests, schemas, maintained docs) is the present system. A task's snippets are contracts or examples; the code to change is whatever is in the repository now.
- `results/` records what this execution changed, verified, deviated from, or could not finish. It is the only place runtime facts are written.

**Not for:** work that has no `.plans/<date>-<feature-name>/` directory yet. Settle the frame with `plan-discuss` and write it with `plan-write` first.

## 1. Validate

Read `00-overview.md` in full, then every task file in numeric order, and build the inventory: for each task its `Task-ID`, `Depends-on`, `After`, `Write-scope`, `Produces`, `Acceptance`, and the status of `results/<Task-ID>.md` if one exists. The inventory is rebuilt from the files on every start and is not stored.

Check:

1. Every task has all six fields, and Task-IDs are unique.
2. Every `Depends-on` file exists and is non-empty.
3. Every `After` names an existing task, the graph has no cycle, and numeric order is a topological order.
4. Every acceptance ID a task cites exists in `00-overview.md`, and every ID in the overview is cited by at least one task.
5. Write scopes that overlap, or that share an indirect output such as a manifest or generated file, belong to tasks ordered by `After`.

When a check fails:

- **No `results/00-execution.md` yet:** the plan is still a draft. Do not freeze it. Report the defects to the user and fix them in the plan with their agreement, then validate again.
- **Already frozen:** record the defects under `Plan defects` in `results/00-execution.md`, set the state to `blocked`, and report. Do not repair the authored files.

## 2. Freeze

Only after validation passes, and before any task changes anything, create `results/00-execution.md`:

```markdown
# Execution

- Plan: .plans/2026-03-10-session-expiry
- Started: <date and time, when available>
- State: executing
- Successor plan: none

## Plan defects
- None.

## Final acceptance
Filled in at the end.
```

`State` is one of `executing`, `blocked`, `completed`, `superseded`.

The file's existence freezes the plan. From here on, never edit the design files or `tasks/`, not to fix them, not to add a requirement, and not afterwards to make them match what was built. If the file already exists, the plan is frozen and this run is a resume.

## 3. Run the task graph

A task is **ready** when every Task-ID in its `After` has a valid `completed` result. Recompute the ready set after every task.

On resume, a `completed` result counts only if it is valid (see section 5) and its verification still holds against the current repository. When the repository may have changed since, re-run the verification and record the outcome as a new attempt. Missing, `blocked`, `failed`, or invalid results satisfy nothing.

**Sequential** is the complete behavior: take the lowest-numbered ready task, run it as the worker, validate its result, recompute.

**Parallel** is optional, when sub-agents are available. Delegate several ready tasks at once only when their write scopes and side effects are clearly disjoint. Shared manifests, generated files, migrations, fixtures, caches, repository-wide formatting, or any doubt mean serializing. Each sub-agent gets exactly one task.

A sub-agent has not read this skill. Its brief contains:

- The plan directory path and the path of its task file.
- The worker protocol in section 4, in full.
- The result template in section 5 and the path `results/<Task-ID>.md`.
- That it must not edit plan files, touch anything outside `Write-scope`, or take on other tasks.

When no task is ready and unfinished tasks remain, stop and report which dependency, blocker, or invalid result is in the way. Do not go around the graph.

## 4. Worker protocol

The worker is a sub-agent, or the coordinator handling one task. For every task, including a resumed or trivial one:

1. Read the whole task file and every file in its `Depends-on`, now. Having read them for an earlier task does not count.
2. State the design constraints that bind this task: boundaries, interfaces, technology, scope.
3. Inspect the current code, tests, and schemas at the files and symbols the task names.
4. Implement the declared outcome inside `Write-scope`. If the outcome cannot be reached without leaving it, stop (see section 6).
5. Run the task's verification and its "Verify against the design" checks.
6. Write `results/<Task-ID>.md`.
7. Return the result path and status. The file is the report; do not replace it with a long message.

## 5. Task results

One file per task:

```markdown
# TASK-02 Result

- Task-ID: TASK-02
- Attempt: 1
- Status: completed

## Changed files
- `src/auth/session.ts`

## Verification
- Ran: `<command or scenario>`
- Outcome: `<exit status and the relevant output>`

## Acceptance evidence
- AC-03: `<what was observed>`

## Design conformance
- Architecture boundaries: pass, `<evidence>`
- Interface contracts: pass, `<evidence>`

## Deviations
- None.

## Blockers
- None.

## Follow-up plan
- Required: no
```

| Status | Meaning | Next |
|---|---|---|
| `completed` | The outcome is in place and all checks pass | Satisfies `After` |
| `failed` | The work was attempted and verification does not pass; the cause is in the implementation | Retry |
| `blocked` | The task cannot proceed without a decision or a change outside its authority | Section 6 |

A retry increments `Attempt` and rewrites the sections above for the new attempt. Move the previous attempt to an `## Earlier attempts` section at the end of the file, as a few lines saying what was tried and why it did not pass. After two `failed` attempts at the same task, stop retrying it and report to the user.

The coordinator validates every result, its own included:

1. Read the file; do not rely on the worker's message.
2. Check the Task-ID, that every changed file is inside `Write-scope`, that verification outcomes are passes with evidence, that each cited acceptance ID has evidence, and that no deviation or blocker contradicts the status.
3. Reject a `completed` result that fails any of these, and have it corrected or the task retried. Do not edit a worker's technical conclusions into a more convenient story.

A deviation is a difference from the task text that leaves every contract intact. Anything that changes a contract is a blocker, not a deviation.

## 6. Conflicts and new requirements

Stop the affected task when reaching its outcome would require changing a non-goal, a technology decision, a module boundary, a public interface, a task dependency, the meaning of an acceptance criterion, or a write scope.

1. Write a `blocked` result with the repository evidence and the decision that is needed, and `Follow-up plan: Required: yes` with the reason.
2. Report to the user. If the agreed frame has to change, that is settled with `plan-discuss`; the change is then written with `plan-write` as a separate `follow-up` or `corrective` plan.
3. Once a successor plan exists, record its path in `results/00-execution.md`, and set the state to `superseded` if the successor takes over the remaining work.

A requirement the user adds during or after execution takes the same route. It is never appended to a task or to the acceptance criteria.

Tasks that do not depend on the blocked one may continue, unless continuing would prejudge the decision the user has to make.

## 7. Final acceptance

When every task has a valid `completed` result:

1. Check each acceptance ID in `00-overview.md` against the result evidence and the repository as it is now, and run any final verification the plan specifies.
2. Confirm every changed file falls inside some task's `Write-scope` and everything listed under `Produces` exists.
3. Fill in `Final acceptance` in `results/00-execution.md`: one line per acceptance ID with its evidence and whether it is met, then any deviations that remain.
4. Set the state to `completed` only when every criterion is met. Otherwise set `blocked` and say what is missing.

Report the final state to the user, with whatever is blocked or deviates and the path to `results/`.

Before any commit, confirm that `.plans/` is covered by the repository-root `.gitignore`, and keep it out of the commit unless the user asked for versioned plan history.
