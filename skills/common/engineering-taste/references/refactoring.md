# Refactoring judgment

This file expands the smell triage in `SKILL.md`. It owns behavior-preserving discipline, smell diagnosis, and named moves. Multi-move sequences live under `recipes/`.

## Discipline

Refactoring changes internal structure without changing observable behavior so the next change is easier and safer.

- Deliberately switch between behavior work and structural work.
- Move in small verifiable steps; keep code runnable.
- Leave touched code slightly healthier.
- Notice the second repetition; refactor the third.
- Add flexibility only for current or evidenced near-term needs.
- Optimize from measurement, except under explicit hard real-time constraints.

## Smell triage

| Smell | Symptom | First candidates |
|---|---|---|
| Mysterious Name | Reader must inspect implementation | Rename; Change Function Declaration |
| Duplicated Code | Same behavior in several places | Extract; Parameterize; Pull Up |
| Long Function | Several intentions/phases in one body | Extract; Split Phase; guards; Split Loop |
| Long Parameter List / Data Clump | Same values/dependencies travel together | Whole Object; Parameter Object; cohesive object |
| Global/Mutable Data | Updates are dispersed or temporal | Encapsulate; narrow scope; query derived state |
| Divergent Change | One module changes for unrelated reasons | Extract owner; Split Phase; Move Function |
| Shotgun Surgery | One rule change touches many modules | Move behavior/data into one owner |
| Feature Envy | Function uses another owner more than its own | Extract then Move Function |
| Primitive Obsession | Raw string/number repeats policy | Replace Primitive with Object |
| Repeated Switches | Same variant axis branches repeatedly | Owned dispatch; polymorphism recipe |
| Lazy Element / Middle Man | Wrapper adds no intention or boundary | Inline; remove middleman |
| Speculative Generality | Unused hooks/factories/protocols | Remove dead code; collapse abstraction |
| Large Class / God Module | No obvious home for new behavior | God-module recovery recipe |
| Data Class | Callers manipulate passive bag and invariants | Move behavior to owner; immutable DTO only at boundaries |
| Fake Modularity | More files, same dependency hairball | Reassign ownership; Split Phase |
| Class Theater | Construction/self with no state, dependency, invariant, lifecycle, or protocol | Free function or real object boundary |
| Comment Deodorant | Comments compensate for tangled structure | Improve structure; retain why/order/invariants |

## Common moves

| Move | Use when | Guard |
|---|---|---|
| Extract Function | Block has one nameable intention | Do not fragment critical order or ferry excessive context |
| Inline Function/Class | Element adds no intention/boundary | Keep real policy/isolation/substitution boundaries |
| Introduce Parameter Object | Values share meaning/invariants | Do not group one-time coincidence |
| Combine Functions into Class | Functions share dependencies/state/invariants | Keep independent pure calculations free |
| Split Phase | Contexts communicate through clear handoff data | Avoid when interaction is densely bidirectional |
| Extract/Move Class or Module | Cohesive behavior has a separate owner | Do not split only for size |
| Replace Primitive with Object | Domain validation/policy repeats | Avoid empty wrappers around raw values |
| Guard Clauses / Decompose Conditional | Exceptional paths hide intent | Preserve cleanup and ordered side effects |
| Replace Conditional with Polymorphism | Same variant controls multiple behaviors | One local switch may be clearer |
| Separate Query from Modifier | Read has surprising side effects | Keep truly indivisible transactions together |
| Replace Inheritance with Delegation | Hierarchy exists for reuse, not substitution | Keep real subtype contracts |
| Remove Dead Code | No current user or registration | Confirm reflective/config/runtime uses |

Use `recipes/split-phase.md`, `recipes/replace-conditional-with-polymorphism.md`, or `recipes/recover-god-module.md` when several moves must be sequenced.
