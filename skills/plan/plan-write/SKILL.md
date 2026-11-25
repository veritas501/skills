---
name: plan-write
description: Write an agreed multi-step change as a layered plan directory of design and task files.
---

# plan-write

## Overview

Write an agreed change as a **layered Markdown plan**: stable design files plus bounded task files that a coordinator can execute without rediscovering architecture or receiving pre-written implementation.

Keep three artifacts distinct:

- **Plan:** point-in-time contract for one agreed change.
- **Results:** execution evidence—changed files, verification, deviations, blockers, and completion state.
- **Current system documentation:** current code, tests, schemas, ADRs, and maintained product documentation.

**Core principle:** Separate stable decisions from task detail, then preserve the authored plan after execution begins. A historical plan explains the intended change; it is not living documentation for the repository forever.

**Companion skills:** `plan-discuss` settles the frame breadth-first. `plan-write` records that frame. `plan-execute` freezes and executes it while re-reading declared design dependencies.

## Precondition: agreed frame required

`plan-write` turns an **already-agreed** frame into files. It is not where the approach is selected.

```
NO PLAN FILES BEFORE THE FRAME IS AGREED.
```

Before writing, confirm goal, non-goals, architecture, technology, and acceptance with the user. If requirements remain fluid or materially different approaches are open, stop and load `plan-discuss` first.

Pipeline order is fixed: **plan-discuss → plan-write → plan-execute**.

## Plan lifecycle and history

### Draft

A plan is editable only before execution starts. While no `results/00-execution.md` exists, revise the design and tasks with the user as needed.

### Frozen

`plan-execute` creates `results/00-execution.md` before the first task. Its presence freezes authored files: `00`–`50` and `tasks/` MUST NOT be edited. Runtime facts belong only in `results/`.

### Follow-up or corrective change

After freeze, a new requirement, changed meaning, invalid design assumption, or material correction requires a separate plan:

- `original`: first plan in a lineage; parent is `none`.
- `follow-up`: additional or changed behavior after an earlier plan.
- `corrective`: repairs a material defect in an earlier agreed design.

The new plan MUST inspect the current repository first. Reference the earlier plan only for lineage and rationale, describe the delta, and restate constraints still needed for safe execution. Never synchronize an old plan to current code or treat it as current architecture documentation.

## Layered authoring

Stabilize the top before drilling down:

1. Write `00-overview.md` and structural dimension files first; obtain user agreement before task files.
2. Probe goal, necessity, solvability, verifiability, and reversibility before expanding a branch into tasks.
3. While still draft, fold task detail back when a top-level decision changes instead of preserving detail based on an obsolete frame.

## Directory structure

```text
.plans/<MM-DD>-<feature_name>/
  00-overview.md        # goal, lineage, constraints, acceptance, assumptions
  10-tech-stack.md      # retained/new technology and rationale
  20-architecture.md    # modules, owners, boundaries, dependencies, data flow
  30-data-model.md      # data structures, storage, schema; omit if N/A
  40-interfaces.md      # public/inter-module contracts; omit if N/A
  50-risks.md           # risks, open questions, blockers
  tasks/
    task-01-<slug>.md
    task-02-<slug>.md
  results/              # created/updated by plan-execute, not authored as task design
    00-execution.md
    <Task-ID>.md
```

Numbered prefixes define read order and stability. `00-overview.md`, `20-architecture.md`, and `tasks/` are required; add or omit other dimensions only when the change needs them.

## Git handling

Plan directories are local coordination artifacts by default. Before creating `.plans/`, determine whether the workspace is a Git repository. If it is, proactively create or update the repository-root `.gitignore` so it contains `.plans/`; do not wait for the user and do not rely only on careful staging.

The `.gitignore` change is normal repository configuration and may be committed. Do not stage or commit files under `.plans/` unless the user explicitly requests versioned plan history.

## Top-level file rules

- `00-overview.md` MUST contain `Plan lineage`, Goal, Non-goals, Constraints, stable Acceptance criteria IDs, and Assumptions.
- `Plan lineage` MUST name **Plan type**, **Parent plan**, and **Trigger**. Original plans use parent `none`.
- Every acceptance ID MUST map to at least one task.
- `20-architecture.md` MUST state module ownership, boundaries, dependency relations, and data flow.
- Top-level files state decisions and contracts, not commands, full code, or step-by-step implementation.
- Durable current-state architecture belongs in maintained docs, ADRs, schemas, tests, and code—not in historical plans.

## Task graph and coordination metadata

Every new task MUST declare:

- `Task-ID`: stable identity such as `TASK-01`.
- `Depends-on`: top-level design files the worker must re-read before the task.
- `After`: predecessor Task-IDs requiring valid completed results; `none` when independent.
- `Write-scope`: exact files or conservative directory patterns the task may modify.
- `Produces`: expected files or durable outputs.
- `Acceptance`: stable acceptance IDs this task helps satisfy.

`Depends-on` is a design-reading gate; `After` is an execution dependency. They are not interchangeable.

Task numbers MUST form a valid topological order and the graph MUST have no cycles. A task becomes ready only after every `After` predecessor completes. Ready tasks may run concurrently only when write scopes and side effects are clearly disjoint. Shared manifests, generated outputs, migrations, fixtures, or uncertain scope force serialization. This declarative graph is sufficient; do not add runtime orchestration code.

## Task file structure

Use contract-level prose, not a replacement implementation:

```markdown
# Task 02: Add session-expiry handling

Task-ID: TASK-02
Depends-on: 00-overview.md, 20-architecture.md, 40-interfaces.md
After: TASK-01
Write-scope: src/auth/session.ts, tests/auth/session.test.ts
Produces: src/auth/session.ts, tests/auth/session.test.ts
Acceptance: AC-03, AC-04

## Constraints from top-level
- Session policy remains owned by the auth module.
- Preserve the existing public session interface and error taxonomy.

## Files and symbols
- Modify `validateSession` in `src/auth/session.ts`.
- Extend expiry scenarios in `tests/auth/session.test.ts`.

## Steps
- Describe expired, valid, and malformed-session behavior, including boundary timestamps.
- Add a failing scenario for the observable expiry contract, then implement within the named owner.
- Run the repository's focused auth test command; expect the new and existing scenarios to pass.

## Verify against top-level
- Confirm ownership and dependency direction match `20-architecture.md`.
- Confirm the public contract still matches `40-interfaces.md`.
- Confirm no technology outside `10-tech-stack.md` was introduced.
```

Prefer stable files, modules, and symbol names over brittle line ranges.

## Minimum sufficient specificity

A task must be executable without guessing its objective or boundaries, but MUST NOT ordinarily pre-write the implementation.

Require concrete:

- Outcomes and observable acceptance behavior.
- Architectural boundaries, ownership, and invariants.
- Public interfaces or schema shapes when relevant.
- Target files, modules, and symbols.
- Edge cases, compatibility constraints, verification commands, and expected outcomes.
- Explicit non-goals and forbidden scope when ambiguity is plausible.

Normally forbid:

- Full function, class, or file bodies.
- Full test bodies.
- “Replace this file with the exact content below.”
- Exact line numbers when a stable symbol or structural location exists.
- Premature private implementation choices such as helper layout or local control flow.

A short code-shaped example is allowed only when the text itself is the contract: a public signature, schema/protocol payload, migration shape, security invariant, or genuinely non-obvious algorithm. Keep it minimal and state why prose would be less precise.

Vague instructions remain plan failures. Removing code does not permit “implement error handling,” “add tests,” or “TBD.” Correct vagueness with precise behavior, boundaries, named locations, edge cases, examples, and verification—not ordinary full source or test bodies.

## Execution handoff

Before invoking `plan-execute`:

1. Confirm the plan is still a draft and the user approved the top-level frame.
2. Validate task metadata, dependencies, write scopes, outputs, and acceptance coverage.
3. Tell the user where authored plan files live and that execution will freeze them by creating `results/00-execution.md`.
4. Invoke `plan-execute`; do not edit the authored plan after freeze.

## Self-review

Before handoff, check:

1. Every acceptance ID maps to at least one task.
2. Every task has `Task-ID`, `Depends-on`, `After`, `Write-scope`, `Produces`, and `Acceptance`.
3. Numeric order is topological; dependencies are valid and acyclic; overlapping scopes are serialized.
4. Every task restates the top-level constraints it must honor.
5. No implementation detail leaked into top-level files.
6. No ordinary full source/test body or exact-content replacement appears in tasks.
7. Task prose remains specific about outcomes, boundaries, symbols, invariants, edge cases, and verification.
8. `Plan lineage` exists and correctly identifies original, follow-up, or corrective history.
9. The plan is still a draft; no execution marker exists.
10. No later requirement is being folded into an already-frozen plan.

Fix defects before handoff.

## Common mistakes

| Mistake | Fix |
|---|---|
| Writing tasks before the frame is agreed | Return to `plan-discuss`; approve `00` and structural files first. |
| Omitting coordination metadata | Add all six stable fields; do not make the executor infer dependencies or scope. |
| Editing an old plan to match later code | Preserve it as history; inspect current state and create a linked follow-up/corrective plan. |
| Treating a plan as current architecture documentation | Update maintained docs/ADRs/schemas when current-state documentation is needed. |
| Writing full source or test bodies in a task | State contracts, target symbols, behavior, invariants, scenarios, and expected verification instead. |
| Becoming vague after removing code blocks | Add observable outcomes, edge cases, named locations, and pass/fail evidence. |
| Using exact line numbers | Name the stable symbol or structural location. |
| Parallelizing overlapping or uncertain scopes | Serialize them with `After`; parallelism requires clearly disjoint writes and side effects. |
