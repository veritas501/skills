# Recipe: Recover a God Module

Use for multi-thousand-line modules or function seas with unclear ownership. This is incremental recovery, not a rewrite.

## Preconditions

- Identify public entrypoints and characterize their observable behavior.
- Map callers, shared state, dependency directions, and reasons to change.
- Do not begin by inventing folders or a universal service class.

## Sequence

1. Add the thinnest behavior/golden checks around public entrypoints.
2. Rename unclear symbols and remove dead local noise.
3. Identify change axes: IO, parsing, domain policy, state/lifecycle, rendering, integration.
4. Extract pure, nameable calculations when this reduces context.
5. Split phases where a clear handoff structure exists.
6. Group repeated dependencies/invariants into a cohesive object when justified.
7. Move behavior next to the data/policy it uses most.
8. Collapse one-shot or forwarding-only classes.
9. Remove duplicate paths after all callers use the new owner.
10. Split files only after the ownership boundary is real.

## Invariants

- Every step leaves the system runnable and verifiable.
- Public behavior does not change during structural moves.
- Dependency direction becomes simpler; no new cycles.

## Stop when

New behavior has an obvious owner, public entrypoints are thin, and a change no longer requires understanding the whole module.

The entry router selects a matching example separately; this recipe does not depend on example files.
