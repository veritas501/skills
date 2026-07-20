# Recipe: Replace Conditional with Polymorphism

Use when the same variant/type axis controls behavior in multiple functions or modules. Do not replace a single clear switch merely to look object-oriented.

## Preconditions

- At least two behaviors branch on the same variants, or adding a variant requires editing several switches.
- Variants have stable semantic identity.
- Tests cover every variant and the unknown/default case.

## Sequence

1. Move variant-dependent behavior behind one calculator/strategy boundary.
2. Introduce a factory or owned dispatch point while preserving the old calls.
3. Create one implementation per real variant.
4. Move one branch at a time; test after each move.
5. Put common default behavior on the base/delegate only when most variants share it.
6. Delete the repeated switches after every caller uses the owned boundary.

## Invariants

- One variant selection point owns dispatch.
- Callers depend on behavior, not type codes.
- Unknown types still fail explicitly.

## Stop when

Adding one variant requires one implementation plus one explicit registration/dispatch change, not edits across unrelated modules.

The entry router selects a matching example separately; this recipe does not depend on example files.
