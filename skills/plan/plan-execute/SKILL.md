---
name: plan-execute
description: Use when executing or resuming a layered plan-write directory, especially when tasks have dependencies, bounded write scopes, durable results, optional sub-agent delegation, or risk of architectural drift and retroactive plan editing.
---

# plan-execute

## Overview

Coordinate one layered plan from freeze through acceptance. Top-level design files remain a **hard gate** for the change being executed; workers also inspect the current repository and implement the declared outcome rather than blindly copying task snippets.

Keep three authorities separate:

- The frozen plan records the agreed change and its boundaries.
- Current code, tests, schemas, ADRs, and maintained docs describe the present system.
- `results/` records what this execution changed, verified, deviated from, or could not complete.

**Core principle:** Re-read design before every task, execute only ready work within declared scope, and preserve history. Runtime facts extend results; they never rewrite the authored contract after execution begins.

## When to use

- Executing or resuming `.plans/<MM-DD>-<feature_name>/`.
- Coordinating dependency-aware tasks with bounded write scopes.
- Using optional sub-agents without making them autonomous planners.
- Recovering from drift, stale results, a changed requirement, or a material design conflict.

## Iron precondition

```
NO TASK STARTS BEFORE READING ITS Depends-on FILES.
```

For every task, including a resumed or apparently trivial one, read the complete task and re-read every declared `Depends-on` file. Memory is not a substitute for the authored gate.

The plan is authoritative only for the change it records. Before editing, inspect the current repository at the task's named files, modules, and symbols. Treat snippets as contract examples, not copy-paste source unless the task explicitly identifies a short contract-defining shape.

## Execution freeze

Before dispatching or implementing the first task:

1. Read `00-overview.md` fully and retain its Non-goals, Constraints, and stable Acceptance IDs.
2. Inventory task files in numeric order.
3. Create `results/` if absent.
4. Create `results/00-execution.md` before any task changes, recording:
   - Plan path.
   - Execution start time when available.
   - State: `executing`.
   - Successor-plan path when later known.

Presence of `results/00-execution.md` freezes `00`–`50` and `tasks/`. From then on:

- Do not edit authored plan files.
- Write runtime state only under `results/`.
- If the marker already exists, treat the plan as frozen and resume from its inventory and task results.
- State may later become `completed`, `blocked`, or `superseded`; authored files remain unchanged.

## Inventory and validation

For every task, record:

- Filename and numeric order.
- `Task-ID`.
- `Depends-on` design files.
- `After` predecessor Task-IDs.
- `Write-scope`.
- `Produces`.
- `Acceptance` IDs.
- Existing `results/<Task-ID>.md` status and validity.

Before deriving work:

1. Confirm Task-IDs are present and unique.
2. Confirm every `Depends-on` file exists and is non-empty.
3. Confirm every `After` reference names a task and the graph has no cycle.
4. Confirm numeric order is a valid topological order.
5. Confirm every Acceptance reference exists in `00-overview.md` and every overview Acceptance ID maps to at least one task.
6. Review write scopes conservatively for overlap and indirect shared outputs.

A defect in a metadata-rich frozen plan is a blocker. Record concrete evidence in results; do not silently repair the task or top-level files.

## Resume rules

- Skip a task only when its result is a valid `completed` result and its verification still establishes the contract against the current repository.
- Missing, `blocked`, `failed`, invalid, or stale results do not satisfy dependencies.
- A valid completed result has the correct Task-ID, stays within `Write-scope`, reports passing task and top-level checks, and has no unresolved deviation or blocker contradicting completion.
- If repository changes may have made evidence stale, re-run the relevant verification. Record the new attempt or outcome in the same result file; do not alter the plan.

## Ready set and execution mode

A task is ready only when every Task-ID in `After` has a valid completed result. Recompute the ready set after every task or parallel wave.

### Optional parallel mode

- A sub-agent receives exactly one task.
- Delegate multiple ready tasks only when their `Write-scope` values and side effects are clearly disjoint.
- Shared manifests, generated files, migrations, fixtures, caches, global formatting, or uncertain side effects force serialization.
- The coordinator retains acceptance ownership; workers may not invent tasks, change the agreed frame, or expand scope.

### Sequential fallback

If sub-agents are unavailable, unsupported, or unsafe, execute the same graph sequentially: choose the lowest-numbered ready task, complete and validate its result, then recompute readiness. Sequential operation is complete behavior, not degraded semantics.

If no task is ready and incomplete tasks remain, report the dependency, blocker, or invalid-result evidence. Do not bypass the graph.

## One-task worker protocol

A worker is either one spawned sub-agent or the coordinator operating in single-task mode. It MUST:

1. Read the complete assigned task and every `Depends-on` file.
2. Restate the concrete top-level, interface, technology, scope, and compatibility constraints before editing.
3. Inspect current code, tests, schemas, and maintained docs at the named locations.
4. Implement the declared outcome only within `Write-scope` and preserve owner/module boundaries.
5. Treat task snippets as contracts or examples, not presumed current implementation.
6. Run the task's verification and its “Verify against top-level” checklist.
7. Write or update `results/<Task-ID>.md` before returning.
8. Return the result path and status. Do not replace the durable result with a separate long narrative.

## Canonical task result

Use one durable file per task. Increment `Attempt` when retrying the same Task-ID.

```markdown
# TASK-02 Result

- Task-ID: TASK-02
- Attempt: 1
- Status: completed

## Changed files
- `path/within/write-scope`

## Verification
- Command/scenario: `<what ran>`
- Outcome: `<exit/result and relevant evidence>`

## Acceptance evidence
- AC-03: `<observable evidence>`

## Top-level conformance
- Architecture boundaries: pass — `<evidence>`
- Technology constraints: pass — `<evidence>`
- Interface contracts: pass — `<evidence>`

## Deviations
- None.

## Remaining blockers
- None.

## Follow-up plan
- Required: no
- Reason: none
```

Allowed `Status` values are `completed`, `blocked`, and `failed`. A blocked or failed result MUST preserve concrete evidence and may list no changed files. Only a valid `completed` result satisfies `After`.

When applicable, `Follow-up plan` MUST state whether a linked plan is required and why. Do not label a material design conflict as a harmless deviation.

## Coordinator result validation

After each worker returns:

1. Read the result file rather than relying on chat summary.
2. Validate identity, attempt, status, changed-file scope, verification outcomes, Acceptance evidence, top-level conformance, deviations, and blockers.
3. Reject completion when required sections are missing or evidence contradicts the status.
4. Preserve the worker's technical conclusions. Request a corrected result or retry; do not rewrite its conclusions into a more convenient story.
5. Recompute readiness from valid completed results.

Independent ready work MAY continue when another task is blocked, but only if dependencies and write scopes remain safe.

## Material conflict or new requirement

Stop the affected task when implementation reality would require changing a Non-goal, technology decision, module boundary, public interface, task dependency, acceptance meaning, or write scope.

Then:

1. Write a `blocked` result with the exact repository evidence and unresolved decision.
2. Preserve all existing results and do not edit the frozen plan.
3. Use `plan-discuss` when the agreed frame must change or is not already explicit.
4. Use `plan-write` to create a separate linked `follow-up` or `corrective` delta plan.
5. Inspect the current repository when authoring and executing the new plan; use the old plan only for lineage and rationale.
6. Optionally set the old execution state to `superseded` and record the successor path in `results/00-execution.md`.

A new user requirement during or after execution follows the same path. Never append it to old tasks, rewrite old Acceptance criteria, revise historical architecture to describe later code, or assume the old plan describes current state.

The coordinator may continue unaffected independent work only when doing so cannot prejudge the required user decision or conflict with the successor plan.

## Final acceptance

After all tasks have valid completed results:

1. Re-check every Acceptance ID in `00-overview.md` against result evidence and the current repository.
2. Run the plan's final verification scenarios where specified.
3. Confirm all changed files are covered by task write scopes and all produced outputs exist.
4. Update maintained docs, ADRs, or schemas only when a task requires those current-state artifacts.
5. Record actual changed files, verification, deviations, blockers, and final state in `results/`.
6. Set execution state to `completed` only when every acceptance criterion is met.

Do not back-sync plans: never edit the historical authored plan merely to make its prose resemble the final private implementation. If behavior or an agreed contract changed materially, use a linked plan instead.

## Legacy Plan Compatibility

An older plan may contain only `Depends-on`, `Files`, Steps, and verification text. Execute it conservatively without retroactive conversion:

- Derive a temporary task identity from the filename.
- Execute tasks in strict numeric order.
- Disable parallelism.
- Infer allowed changed files from its `Files` section.
- Re-read declared design files before each task.
- Write result files when practical using the canonical sections.
- Do not rewrite the legacy plan to add new metadata, lineage, or freeze semantics.

If legacy scope or dependencies are ambiguous, serialize and ask rather than infer a risky boundary.

## Drift red flags

- “I will update the old plan so it reflects the new implementation.” → Preserve history; record facts in results.
- “The user added a requirement, so I will append it to the existing plan.” → Create a linked delta plan.
- “The task contains code, so I can paste it without inspecting the repository.” → Inspect current symbols and treat snippets as contracts/examples.
- “The private implementation differed, so I should revise the plan afterward.” → Record the as-built evidence; use a new plan only for material contract change.
- “The task is ready enough even though one predecessor is blocked.” → It is not ready; honor `After`.
- “The scopes probably do not overlap.” → Uncertainty requires sequential execution.

## Common mistakes

| Mistake | Fix |
|---|---|
| Reading `00-overview.md` once | Re-read every task's complete `Depends-on` set before that task. |
| Starting work before the freeze marker | Create `results/00-execution.md`, then treat authored files as immutable. |
| Trusting a completed label without evidence | Validate the canonical result and re-run stale verification. |
| Giving a sub-agent several tasks | One worker receives exactly one bounded task. |
| Replacing worker evidence with a coordinator summary | Keep the result file authoritative; validate or request correction. |
| Silently expanding write scope | Stop, record the blocker, and route material change to a linked plan. |
| Requiring sub-agents | Use the complete sequential fallback. |
| Modernizing a legacy plan during execution | Execute conservatively; do not convert or rewrite it. |
