# Plan

Three repository-hosted Skills turn an agreed change into a reviewable, executable, and durable Markdown history—without a separate orchestration service or parser.

## Lifecycle

1. **[plan-discuss](./plan-discuss/SKILL.md)** freezes the frame for one change through breadth-first discussion: goal, non-goals, architecture, technology, and acceptance.
2. **[plan-write](./plan-write/SKILL.md)** records that change as a layered plan with stable task metadata, dependency and write-scope boundaries, acceptance mapping, and plan lineage. Tasks use **minimum sufficient specificity**: concrete behavior, interfaces, invariants, target symbols, edge cases, and verification intent without routine full implementation or test bodies.
3. **[plan-execute](./plan-execute/SKILL.md)** creates `results/00-execution.md` to freeze the authored plan, coordinates ready tasks, and stores changed files, verification, deviations, blockers, and completion evidence in durable result files.

`plan-execute` may delegate clearly independent ready tasks to optional single-task sub-agents. Sub-agent support is never required: the same dependency graph has a complete sequential fallback, and overlapping or uncertain write scopes are serialized.

## Plans are historical change records

An executed plan is not kept synchronized with the current repository. Its `00`–`50` design files and `tasks/` preserve what was agreed when execution began; `results/` preserves what execution actually established.

New requirements or material design corrections create a linked **follow-up** or **corrective** plan. They do not rewrite the frozen parent plan. Each new plan first inspects the current repository and uses prior plans only for lineage and rationale.

Current code, tests, schemas, ADRs, and maintained documentation describe the current system. Update those current-state artifacts when a task requires it; do not repurpose historical plans as living documentation.

This protocol remains entirely Skill- and Markdown-based. Safe sub-agent coordination is optional, sequential execution is complete, and retroactive plan editing is prohibited.
