# Workflow details

This file expands only the ambiguous parts of the `SKILL.md` spine: judging local precedent, setting scope, escalation, and matching verification to risk.

## Is the nearest precedent healthy?

A nearby implementation is a useful precedent only when it plays the same system role and has:

- an owner consistent with its data and policy
- explicit dependency and side-effect boundaries
- error handling at the same layer
- tests that express comparable contracts
- no known smell that the requested change would amplify

Similarity of filenames, naming, or call shape alone is not enough. If the nearest pattern is unhealthy, preserve observable contracts and improve only the directly blocking boundary.

## Research depth

Read beyond the changed lines until you can explain:

- who calls the behavior and what they observe
- which downstream operation owns side effects
- where input becomes trusted domain data
- where errors change meaning or become user-visible
- which invariant would be violated by a partial update
- which tests would catch the plausible regression

Stop researching when these questions are answered; do not map unrelated subsystems.

## Scope test

A local refactor belongs in the change when it is necessary to avoid:

- a second behavior path for the same rule
- adding another responsibility to an already mixed owner
- extending an argument/dependency hairball
- exposing internals through a new caller
- preserving a boundary that makes the requested behavior untestable

It does not belong when it only standardizes unrelated code, renames broad surfaces, or cleans modules the requirement does not touch.

## Escalation

Before implementing a public API, dependency, configuration mechanism, architectural pattern, or cross-module migration, state:

1. why existing structure cannot carry the requirement honestly
2. affected callers/configuration/operations
3. migration and compatibility impact
4. why the added surface is current rather than speculative

## Risk-matched verification

Choose evidence by failure mode:

| Risk | Evidence |
|---|---|
| Pure internal restructure | Existing behavioral tests + focused smoke |
| New observable branch | Contract test for success, boundary, and failure |
| Public API/signature | All callers + compatibility/serialization checks |
| State or lifecycle | Transition/invariant tests + cleanup/error path |
| Integration/IO | Real adapter or transport smoke where feasible |
| Performance-motivated structure | Measurement before and after |

A test is useful only if a plausible regression of the intended behavior makes it fail. After checks, inspect the complete diff in module context and update stale comments/docs.
