---
name: plan-write
description: Write an agreed multi-step change as a layered plan directory of design and task files.
---

# plan-write

Record an agreed change as a layered Markdown plan: design files that hold the stable decisions, and bounded task files that an executor can carry out without rediscovering the architecture and without being handed pre-written code.

A plan is a point-in-time contract for one change. It is not documentation of the current system; code, tests, schemas, ADRs, and maintained docs are. What execution actually did is recorded under `results/` by `plan-execute`.

## Precondition

The frame must already be agreed with the user: goal, non-goals, architecture, technology, and acceptance. If any of the five is open, or materially different approaches are still being weighed, stop and use `plan-discuss`. The approach is not chosen here.

## Directory

```text
.plans/<YYYY-MM-DD>-<feature-name>/
  00-overview.md        # lineage, goal, non-goals, constraints, acceptance, assumptions
  10-tech-stack.md      # technology kept and introduced, with rationale
  20-architecture.md    # modules, owners, boundaries, dependencies, data flow
  30-data-model.md      # data structures, storage, schema
  40-interfaces.md      # public and inter-module contracts
  50-risks.md           # risks and how each is contained
  tasks/
    task-01-<slug>.md
    task-02-<slug>.md
  results/              # written by plan-execute; never authored here
```

`00-overview.md`, `20-architecture.md`, and `tasks/` are required. Write `10`, `30`, `40`, and `50` only when the change needs them. The date is the day the plan is written.

Plan directories are local by default. In a Git repository, add `.plans/` to the repository-root `.gitignore` before creating the directory; that change may be committed. Do not stage or commit anything under `.plans/` unless the user asks for versioned plan history.

## Order of writing

1. Inspect the current repository at the places the change touches. Earlier plans describe what was once intended, not what is there now.
2. Write `00-overview.md` and the design files, and get the user's agreement on them before writing any task file.
3. Write the task files.
4. While the plan is a draft, a changed design decision means rewriting the tasks that depend on it from the new decision. Do not patch detail written for the old one.

## Design files

`00-overview.md` follows this skeleton. `plan-execute` looks up the acceptance IDs and non-goals here, so keep the headings and the ID format.

```markdown
# <Change name>

## Plan lineage
- Plan type: original | follow-up | corrective
- Parent plan: none | .plans/<directory of the earlier plan>
- Trigger: <what prompted this plan>

## Goal
## Non-goals
## Constraints

## Acceptance
- AC-01: <observable criterion>
- AC-02: <observable criterion>

## Assumptions
```

- Every acceptance ID maps to at least one task.
- `20-architecture.md` states module ownership, boundaries, dependency direction, and data flow.
- Design files state decisions and contracts. Commands, code, and step-by-step implementation belong in tasks.
- A question that is still open is not a risk to list in `50-risks.md`; resolve it with the user before execution, because the plan cannot be edited afterwards.

## Task files

Every task declares six fields:

| Field | Meaning |
|---|---|
| `Task-ID` | Stable identity, `TASK-01`. Matches the number in the filename |
| `Depends-on` | Design files the worker reads before starting the task |
| `After` | Task-IDs that must be completed first, or `none` |
| `Write-scope` | The files, or conservative directory patterns, the task may modify |
| `Produces` | What must exist when the task is done, checked at final acceptance. May be narrower than `Write-scope`, and may name outputs that are not files |
| `Acceptance` | The acceptance IDs this task helps satisfy |

`Depends-on` is about reading and `After` is about ordering; a task usually needs both.

Task numbers form a valid topological order of the `After` graph, which has no cycles. Two tasks may run concurrently only when their write scopes and side effects are clearly disjoint, so give tasks that touch shared manifests, generated files, migrations, or fixtures an `After` relation that serializes them.

```markdown
# Task 02: Add session-expiry handling

Task-ID: TASK-02
Depends-on: 00-overview.md, 20-architecture.md, 40-interfaces.md
After: TASK-01
Write-scope: src/auth/session.ts, tests/auth/session.test.ts
Produces: expiry handling in `validateSession`, with tests
Acceptance: AC-03, AC-04

## Constraints from the design
- Session policy remains owned by the auth module.
- The public session interface and error taxonomy are unchanged.

## Files and symbols
- Modify `validateSession` in `src/auth/session.ts`.
- Extend the expiry scenarios in `tests/auth/session.test.ts`.

## Steps
- A session whose expiry timestamp equals the current time is expired. A malformed session is rejected with the existing malformed-session error, not the expiry error.
- Add a failing scenario for each case, then implement within `validateSession`.
- Run the focused auth tests: `npm test -- auth/session`. The new and existing scenarios pass.

## Verify against the design
- Ownership and dependency direction match `20-architecture.md`.
- The public contract still matches `40-interfaces.md`.
```

### How specific a task is

A task is executable without guessing its objective or its boundaries, and it does not pre-write the implementation.

State concretely:

- The outcome and the observable behavior that proves it.
- The design constraints the task must honor, restated in the task.
- Target files, modules, and symbols. Name a symbol, not a line number.
- Public interfaces or schema shapes when the task touches them.
- Edge cases, compatibility constraints, the verification command, and its expected result.
- What is out of scope, when a worker could plausibly think otherwise.

Leave out full function, class, file, or test bodies, "replace this file with the content below", and private implementation choices such as helper layout or local control flow.

A short code-shaped example is allowed only when the text itself is the contract: a public signature, a schema or protocol payload, a migration shape, a security invariant, or a non-obvious algorithm.

Removing code does not license vagueness. "Implement error handling", "add tests", and "TBD" are plan defects; replace each with the behavior, the named location, the cases, and the check.

## Lifecycle

- **Draft.** Until `results/00-execution.md` exists, the plan is edited freely with the user.
- **Frozen.** `plan-execute` creates `results/00-execution.md` before the first task. From then on the design files and `tasks/` are never edited, including to make them match the code that was eventually written.
- **After freeze.** A new requirement, a changed meaning, or a design assumption that proved wrong gets a separate plan in its own directory: `follow-up` for added or changed behavior, `corrective` for a material defect in the earlier design. It names its parent in `Plan lineage`, describes only the delta, and restates the constraints it still relies on, since its executor does not read the parent.

## Review and handoff

Check the draft before handing it over, and fix what fails:

1. No `results/00-execution.md` exists.
2. `Plan lineage` is filled in and correct.
3. Every acceptance ID maps to at least one task, and every ID a task cites exists in the overview.
4. Every task has all six fields; every `Depends-on` file exists; every `After` names a real task.
5. Numeric order is topological, there is no cycle, and tasks with overlapping or uncertain scopes are serialized.
6. Every task restates the design constraints it must honor.
7. Design files contain no implementation detail; tasks contain no full source or test bodies and no vague steps.

Then tell the user where the plan is and that executing it freezes the authored files. Start `plan-execute` when they give the go-ahead.
