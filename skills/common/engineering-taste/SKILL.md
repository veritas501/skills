---
name: engineering-taste
description: Use when code lacks a healthy local precedent or shows AI-generated structural smells: unclear ownership, long function-heavy modules, fake classes or fake modularity, repeated type switches, forwarding-only wrappers, speculative abstractions, or refactoring decisions caused by those symptoms.
---

# Engineering Taste

This skill supplies the engineering judgment AI often loses without a healthy human codebase to imitate. The spine is: **study → place → shape → implement → verify**.

## Use / do not use

Use when ownership, boundary, or abstraction judgment is genuinely unclear—especially in greenfield, AI-led, or already AI-shaped code.

Do not load it merely because a feature is non-trivial, for language-style lookup alone, or for a mechanical edit. Do not replace a healthy project convention with generic taste.

## Precedence

1. Explicit user requirement and observable behavior.
2. Project context, public APIs, configuration, formatter, linter, and tests.
3. Healthy patterns in the directly affected subsystem.
4. This skill's structural judgment.
5. Generic language-style defaults.

When precedent is unhealthy, name the concrete problem and correct only the directly affected boundary.

## The engineering spine

### 1. Study the system role

Before editing, read the affected module in full context and find one or two nearest implementations. Establish its role, observable contracts, dependency/side-effect boundaries, data ownership, invariants, failures, and tests.

Do not design from the requested lines alone. Details: `references/workflow.md`.

### 2. Place the behavior

Ask where the requirement naturally belongs:

- Which domain object/module owns the rule?
- Which data and policy change together?
- Where should invariants be enforced?
- Can the next related change happen in one place?
- Does the proposed owner have one coherent reason to change?

Design from requirements, contracts, invariants, failures, tests, and measurements—not “more OOP” or “more engineered.”

### 3. Choose the smallest honest shape

| Evidence | Default shape |
|---|---|
| Stateless, independent, composable calculation | Function |
| Shared dependency/state/lifecycle/identity/invariant/coherent capability | Object/class, even if state is immutable |
| Same values travel together and share meaning | Parameter object/record |
| Pure data with no policy | Dataclass/record/struct |
| True substitutable variants | Interface/polymorphism |
| Reuse without substitutability | Composition/delegation |
| One implementation plus hypothetical futures | No ABC, Protocol, factory, or plugin point |
| Type-level construction genuinely using the type | Class method |
| Operation belongs to a real type API but uses neither instance nor type | Static method; otherwise free function |

A class is invalid when it is only a folder, a one-shot stateless instance, an unjustified inner class, or a forwarding wrapper. Full decisions: `references/design.md`.

### 4. Implement a coherent change

- Make the smallest honest working change—not the smallest diff.
- Perform necessary local refactoring when direct appending would preserve the diagnosed smell; do not refactor unrelated modules.
- Split by ownership/reason to change, not line count; explain any new public API, dependency, configuration, or architecture.
- Comments preserve intent, invariants, order, side effects, and failure conditions—not visible statements.

### 5. Verify and report

Protect observable behavior, invariants, boundaries, and failure paths. Run the project's applicable checks and changed behavior/smoke path; inspect the complete diff in module context. Never weaken tests or suppress valid failures to pass. Details: `references/workflow.md`.

## Mature code vs greenfield

| Context | Required behavior |
|---|---|
| Mature healthy subsystem | Match its ownership, naming, error, dependency, and test patterns |
| Mature but locally unhealthy | Preserve contracts; improve the directly blocking boundary without unrelated cleanup |
| Greenfield / AI-led | Establish clear ownership, dependency direction, object boundaries, and thin public APIs yourself |
| User asks “encapsulate” | Identify shared data/dependencies/invariants first; do not respond by inventing random classes/files |

## Smell triage

| Symptom | Likely problem | First direction |
|---|---|---|
| Multi-kLOC function/module sea | No ownership / mixed change axes | `recipes/recover-god-module.md` |
| Same argument bag through many helpers | Data clump or missing capability object | Parameter object or cohesive object |
| Same switch/type branch in several places | Variant logic has no owner | `recipes/replace-conditional-with-polymorphism.md` |
| Files split but every call still ferries the same dependencies | Fake modularity | Reassign ownership; collapse false boundaries |
| Empty/one-shot class for a stateless method | Class theater | Function or justified type-level API |
| Base class/protocol/factory with one implementation | Speculative generality | Delete until evidence exists |

Full diagnosis and moves: `references/refactoring.md`.

## Load only the needed depth

- **Workflow details:** `references/workflow.md`
- **Design decisions:** `references/design.md`
- **Smells and named moves:** `references/refactoring.md`
- **Multi-move recipes:** `references/recipes/`
- **Few-shot routing:** `references/examples/index.md` — choose one nearest example, never load all
- **Optional language sharp edges:** `references/style/<language>.md` (never a trigger by itself)
- **Style provenance:** `references/style/sources.md`

C and Rust style pages are not official Google guides; their source status is explicit in `references/style/index.md`.

## Hard stops

- No line-count-only file split.
- No public abstraction without a current or evidenced near-term use.
- No forwarding-only wrapper without boundary value.
- No preserving a directly blocking bad structure solely to keep the diff small.
- No architecture/dependency/API expansion without explaining necessity and impact.
- No behavior change without risk-matched verification.
- No tests weakened, errors ignored, or checks suppressed to make validation pass.
