# Rust — not a Google styleguide document

There is **no** Rust guide in `google/styleguide`. This file is **idiomatic Rust + rustfmt/clippy** so the agent has a peer to other languages. Do **not** attribute it to Google policy.

## Tooling (non-negotiable)

- `rustfmt` on all edits; `cargo clippy` clean for touched code when practical.
- Edition: match `Cargo.toml` (prefer modern edition already in crate).

## Naming (Rust convention)

| Kind | Form |
|---|---|
| crates/modules/files | `snake_case` |
| types/traits | `PascalCase` |
| functions/methods/vars | `snake_case` |
| constants/statics | `SCREAMING_SNAKE_CASE` |
| type params | short `PascalCase` (`T`, `E`) |

## API / language pins agents miss

| Topic | Pin |
|---|---|
| Errors | `Result` + thiserror/anyhow per crate norms; `?` ; don’t `unwrap()` in library code without proven invariant |
| Panic | for bugs/impossible states, not expected IO/user errors |
| Clone | don’t spray `.clone()` to silence borrowck — restructure lifetimes or use references |
| Visibility | minimal `pub`; prefer `pub(crate)` |
| Traits | don’t introduce trait objects/generics without need |
| Async | don’t hold locks across `.await`; pick runtime already in tree |
| Unsafe | rare, documented safety invariants in `// SAFETY:` |
| Docs | `///` on public items; examples when non-obvious |

## Structure

Module ownership and dependency boundaries are defined in `../design.md`.

- Prefer standard-library and existing crate patterns; do not add a framework the crate does not already need.

## When user wanted “Google style” on Rust

Say clearly: no official Google Rust guide; applying idiomatic Rust + project Clippy. Optionally align **comments/error seriousness** with Google cross-language principles (clarity, no swallowed errors, no global mutable state).
