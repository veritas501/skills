# Design edge cases

`SKILL.md` owns the default shape table. This file covers cases where the obvious answer is ambiguous and supplies tests for deciding without style preference.

## Ownership test

Move behavior toward the owner whose data, policy, and invariants it uses most. A proposed owner is credible when:

- the behavior changes with that owner
- callers need not know the owner's internal topology
- one business-rule change can be made in one place
- the owner retains one coherent reason to change

Do not create an owner solely to reduce the number of parameters or files.

## When immutable state still justifies an object

Mutable state is not required. An object can be honest when it holds stable dependencies, represents identity, manages a lifecycle/resource, enforces a business invariant, or exposes one coherent domain capability.

Counter-test: if all methods can move to module scope without losing an invariant, lifecycle, policy, or meaningful API boundary, the class is probably theater.

## Repeated dependency test

Several functions sharing parameters do not automatically require a class. Ask:

1. Are the dependencies stable across calls?
2. Do the functions jointly enforce one invariant or policy?
3. Would callers benefit from one smaller capability API?
4. Does construction represent a meaningful configured capability rather than a one-shot call?

Use a parameter object for a meaningful data clump; use a class only when behavior ownership is also present.

## Static/class methods

- A class method is justified by alternate construction or type-level policy and must genuinely use the type.
- A static method belongs only on an already meaningful type API.
- If moving it to module scope loses nothing, prefer module scope.

## Inheritance test

Inheritance requires substitutability: callers using the base contract must remain correct for every subtype. If the goal is reuse, optional behavior, or runtime assembly, use composition/delegation.

Repeated type switches justify polymorphism only when the same variant axis controls multiple behaviors. One local, stable switch can be clearer.

## Module boundary test

A split is real when the new modules:

- have distinct reasons to change
- communicate through a small explicit contract
- have one-way dependencies
- can be understood or tested without reopening each other's internals

If every call still ferries the same dependency bag, the split is cosmetic.

## Comments vs extraction

Extract a block when it is an independently nameable responsibility. Keep an ordered flow together and use semantic landmarks when extraction would hide sequencing, split one transaction, ferry large shared context, or increase navigation cost.

Comments document intent, invariants, ordering, side effects, and failure conditions. They do not rescue a boundary that should be redesigned.
