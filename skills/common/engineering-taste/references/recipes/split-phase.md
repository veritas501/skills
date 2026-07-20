# Recipe: Split Phase

Use when one function mixes two contexts that can communicate through a clear intermediate structure, such as load/parse → domain decision → rendering/IO.

## Preconditions

- Current behavior is characterized by tests or a golden/smoke output.
- The phases have different reasons to change.
- Data can cross the boundary without exposing the producer's internals.

## Sequence

1. Identify phase-specific statements and name the boundary data.
2. Create the intermediate structure without changing callers.
3. Enrich it with every value the second phase needs.
4. Move calculations into the first phase.
5. Change the second phase to consume only the intermediate structure.
6. Remove obsolete dependencies from the second phase.
7. Extract each phase into its own function/module only when ownership is clear.

## Invariants

- Output and side effects remain unchanged after every step.
- The renderer/consumer never reopens source data or recalculates producer rules.
- The intermediate structure is explicit and preferably immutable after construction.

## Stop when

Each phase can change independently and the handoff structure is smaller and clearer than the original shared context.

The entry router selects a matching example separately; this recipe does not depend on example files.
