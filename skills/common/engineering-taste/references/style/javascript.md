# JavaScript — Google Style (delta)

Upstream: `jsguide.html`. **Guide is frozen**; Google points new work at **TypeScript**. Still use these pins for plain JS / legacy.

## Modules & structure

- Prefer **ES modules** (`import`/`export`) for new non-Closure code; Closure uses `goog.module`.
- File order: license → `@fileoverview` → imports → implementation; blank line between sections.
- ES `import` paths: include **`.js` extension** when the guide’s ESM rules apply; don’t line-wrap imports (historically exception to column limit).

## Language

| Pin | Detail |
|---|---|
| Equality | `===` / `!==` by default; `== null` only to catch null **and** undefined |
| `var` | Prefer `const`/`let`; don’t use `var` in modern code unless forced |
| `with` | **Banned** |
| `eval` / `Function(string)` | **Banned** (except real loaders) |
| Block-level `function` decl | Avoid; use `const f = () => {}` / function expr |
| Arrays | `[]` not `new Array()` for empty/literal construction |
| `this` | Don’t use for global / accidental call patterns |

## Naming

- `lowerCamelCase` functions/vars; `UpperCamelCase` classes; `CONSTANT_CASE` for true constants.
- Filenames often `lower_snake` or match package convention; be consistent with repo.

## Formatting

- Trust **clang-format** for JS when used at Google; don’t invent a personal brace style mid-file.
- Historical column limit ~80 for much of the guide; repo tooling wins.

## JSDoc / types

- Public APIs: JSDoc types when not using TypeScript.
- If migrating: prefer **TypeScript style guide** over inventing dual systems.

## Agent anti-patterns

```js
// No
if (x == 0) {}           // use === unless nullish intentional
with (obj) { f(); }      // banned
eval(code);              // banned
import './file';         // often should be './file.js' under ESM rules
```
