---
name: agent-deslop
description: Remove AI slop from code, such as plan or chat residue in comments and needless structure.
---

# Agent Deslop

You already know how to simplify code. This skill pins what counts as slop, what must survive, and the evidence required before touching structure.

## Core rule: behavior never changes

Change only how the code is structured and documented, never what it does. Features, edge cases, public signatures, outputs, and side effects stay intact.

Slop is only what can be removed with no observable difference. When a cleanup cannot be shown to be behavior-neutral, leave the code as it is and list it in the report. This rule outranks everything below.

Behavior includes more than control flow: exported names, parameter names callers can pass by keyword, serialized field names, and the text of log and error messages are all part of it.

## Scope

- Work on the module, file, or directory the user names.
- With no scope given, take the changes of the current session: uncommitted work including staged and untracked files, plus commits made in this session. When that set is empty or its extent is unclear, ask for a scope instead of guessing.
- Inside a diff, stay within the changed hunks. Never touch unrelated code.
- Skip generated, vendored, and third-party files.

## Comment slop

Comments and docstrings must read as static documentation for a maintainer who never saw the chat or the plan.

| Kind | Looks like | Action |
|---|---|---|
| Plan and phase leakage | `Phase-6`, `Task 1.2`, `Step 3`, `M1`, `Part B` | Remove the reference |
| Temporal or conversational notes | `as requested`, `updated to`, `previously`, `now supports`, `added for`, `fixed` | Rewrite as present-tense behavior |
| Narration | `// increment counter` above `counter += 1`; a docstring that only repeats the signature | Delete |
| Tombstones | `// removed X`, `// moved to Y`, commented-out old code | Delete |
| Notes to a reviewer | `Note: this is important`, `NOTE: changed from X` | Delete, or rewrite as the fact it stands for |
| Decoration | Banner separators and emoji the surrounding code does not use | Delete |

Rules that keep this from destroying real documentation:

- **Remove a term only when it points outside the codebase**, to a plan or a conversation. `Phase`, `Step 3`, and `previously` are also domain language: compiler phases, two-phase commit, protocol steps, a "previously acquired lock". Those stay.
- **Strip the tag, keep the insight.** When a comment mixes a phase tag with a useful explanation, remove only the tag and smooth the prose.
- **Keep the why.** Design intent, trade-offs, performance constraints, edge cases, concurrency and safety invariants (`SAFETY`, `INVARIANT`, lock ordering), and valid links to docs or contracts are not slop. A `fixed ...` note that explains a workaround becomes a present-tense statement of what the code guards against.
- **Never touch comments that tools read**: `# noqa`, `# type: ignore`, `// nolint`, `// eslint-disable`, `//go:build`, license headers, and TODOs that reference an issue tracker.

## Structural slop

Follow the project's instruction files (`AGENTS.md`, `CLAUDE.md`, or the equivalent) and the surrounding code over generic taste. Prefer guard clauses to deep nesting and explicit code to dense one-liners or clever idioms. Do not merge unrelated concerns or collapse a helpful abstraction to save lines.

Removal needs evidence, never just the appearance of being unused or unreachable:

| Target | Evidence required |
|---|---|
| Dead code | No reference anywhere in the repository, dynamic ones included (lookup by string, registries, reflection, FFI), and the symbol is not exported or otherwise public |
| Redundant defensive check | The condition is ruled out by something enforced: a type the compiler actually checks, or validation on every path into the function. The callers you happened to read, unchecked type hints, and casts are not evidence |
| Pass-through wrapper | Private, with every call site inside the scope being edited. A public wrapper is a signature and stays |
| Rename, including names carrying plan residue such as `foo_v2_fixed` or `test_task_2_1` | A local variable or private symbol whose every use lies inside the scope being edited. Anything exported, serialized, referenced by string, or selected by name in CI keeps its name |

Leftover debug output and scratch files from the session are reported, not silently deleted: removing output is a behavior change.

## Example: what to keep

Deslopped:

```text
Before: Per-build opcode inverse-permutation (Phase-6 polymorphic ISA). Updated as
        requested so that after rolling-key decoding recovers a wire opcode byte,
        one lookup maps it back to the canonical opcode.

After:  Per-build opcode inverse-permutation for the polymorphic ISA. After
        rolling-key decoding recovers a wire opcode byte, one lookup maps it back
        to the canonical opcode.
```

Left alone, although it contains the same trigger words:

```text
Phase 2 of the handshake: the server proves key possession before the client
sends credentials. Previously issued tickets stay valid until they expire.
```

"Phase 2" names a protocol phase and "previously issued" describes runtime state. Neither refers to a plan or a conversation.

## Verify and report

- Comment-only edits need the formatter and linter, not a test run.
- Every structural edit needs the project's build and tests. When no test covers the touched path, treat the edit as unproven: revert it and report it.
- Report the files changed, what was removed by kind, which checks ran with their results, and every cleanup left unapplied together with the evidence that was missing.
