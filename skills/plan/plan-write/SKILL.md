---
name: plan-write
description: Use when writing an implementation plan for a multi-step feature, before touching code, especially when a single plan.md would mix top-level design with implementation detail. Use when you have a spec or agreed design and need to lay it out as files an executor can follow without drifting.
---

# plan-write

## Overview

Write implementation plans as a **layered, multi-file directory** instead of one `plan.md`. Top-level design is split by dimension into separate files; each subtask is its own file that references the top-level files it depends on.

**Core principle:** A single plan.md mixes stable top-level design with volatile implementation detail. The detail buries the design, and executors drift from it. Physical separation by layer and dimension is what gives the top-level design enforcing power.

**Companion skills:** The discussion that produces the agreed frame is governed by `plan-discuss` (breadth-first, probe, fold). Plans written this way are executed with `plan-execute`, which enforces reading the top-level files before any task.

## When to Use

- Writing a plan for any feature that needs more than ~2 tasks
- You have an agreed spec/design and need to turn it into an executable plan
- Past plans drifted: executor ignored architecture, re-chose tech, broke module boundaries
- A plan.md is growing large and the top-level intent is getting buried

**When NOT to use:** trivial single-file changes, throwaway scripts. A directory is overkill there.

## Layered Authoring (breadth-first)

Do NOT write all layers at once. Stabilize the top before drilling down.

1. **L0/L1 first** — write `00-overview.md` (contract) and the structural dimension files (architecture, tech-stack). Get user agreement on these BEFORE writing any task file.
2. **Probe before drilling** — before expanding a branch into task detail, run the five-question feasibility probe from `plan-discuss` (goal / necessity / solvability / verifiability / reversibility). If any fails, stop and surface the blocker — don't write speculative tasks.
3. **Fold when top changes** — if a top-level file changes, compress affected task detail back to a summary rather than rewriting it all. Top-level files are stable; task files are volatile.

## Directory Structure

```
.plans/<MM-DD>-<feature_name>/   # relative to working directory
  00-overview.md        # L0 contract: goal, non-goals, constraints, acceptance, assumptions
  10-tech-stack.md      # language, framework, key deps + versions, why
  20-architecture.md    # modules, boundaries, dependency relations, data flow
  30-data-model.md      # data structures, storage, schema   (omit if N/A)
  40-interfaces.md      # inter-module / API contracts        (omit if N/A)
  50-risks.md           # risks, open questions, blockers
  tasks/
    task-01-<slug>.md
    task-02-<slug>.md
    ...
```

**Numbered prefix = read order = stability.** `00` is most stable (the contract); `50` changes most. Add dimensions per project (`15-ui-design.md`, `25-deployment.md`); drop ones that don't apply. `00-overview.md`, `20-architecture.md`, and `tasks/` are the non-negotiable skeleton.

**Split by responsibility, not layer.** Files that change together live together. One dimension per file so changing architecture never touches tech-stack.

## Top-Level File Rules

Each dimension file is short and declarative — it states decisions, not implementations.

- `00-overview.md` MUST contain: Goal (1 sentence), Non-goals, Constraints, Acceptance criteria, Assumptions. NO tech choices, NO code.
- `20-architecture.md` MUST contain module list + boundaries + dependency relations. NO function-level code.
- Forbidden in any top-level file: full code, command flows, step-by-step implementation. Those go in task files only.

## Task File Structure (anti-drift)

Every task file MUST start with a `Depends-on:` header listing the top-level files it must obey. This is the hook `plan-execute` uses to force re-reading the design before implementing.

```markdown
# Task 01: <component name>

Depends-on: 00-overview.md, 20-architecture.md, 40-interfaces.md

## Constraints from top-level
- <copy the specific architecture boundary / interface contract this task must honor>

## Files
- Create: exact/path/to/file.ext
- Modify: exact/path/to/existing.ext:120-145
- Test: tests/exact/path/test.ext

## Steps
- [ ] Step 1: write failing test  (full test code here)
- [ ] Step 2: run, expect FAIL
- [ ] Step 3: minimal implementation (full code here)
- [ ] Step 4: run, expect PASS
- [ ] Step 5: commit

## Verify against top-level
- [ ] Output respects boundaries declared in 20-architecture.md
- [ ] No new tech introduced beyond 10-tech-stack.md
```

**No placeholders.** Each code step shows the actual code. "TBD"/"implement later"/"add error handling" are plan failures.

## Execution Handoff

Layered discussion is for *thinking clearly*; execution needs full context. Before handing off:

1. Confirm L0/L1 files are frozen (user-approved).
2. Tell the user: tasks live in `.plans/<MM-DD>-<feature_name>/tasks/`, top-level design in `00`–`50`.
3. Invoke `plan-execute` to run tasks task-by-task with top-level files as a hard precondition.

## Self-Review

After writing the directory, check:

1. **Coverage:** every acceptance criterion in `00-overview.md` maps to at least one task.
2. **Depends-on present:** every task file has a `Depends-on:` header and a "Constraints from top-level" section.
3. **No leakage:** no code/commands in `00`–`50`; no design decisions buried only in a task.
4. **Naming consistency:** types/functions named in later tasks match earlier ones.

Fix inline.

## Common Mistakes

| Mistake | Fix |
|---|---|
| Writing all task files before user approves top-level | Freeze `00`/`20` first. Tasks are volatile, design is the anchor. |
| Task file with no `Depends-on:` | Executor has nothing to re-read → drift. Always add it. |
| Code snippets sneaking into `20-architecture.md` | Move to a task file. Top-level states decisions only. |
| One giant `tasks/task-01.md` doing everything | One responsibility per task file. |
