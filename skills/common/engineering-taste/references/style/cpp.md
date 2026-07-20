# C++ — Google Style (delta)

Upstream: `cppguide.html`. Prefer **`clang-format`** with Google style + `cpplint` ecosystem where used.

## Headers

- Self-contained `.h` (compiles alone); include what you use (**IWYU**); don’t rely on transitive includes.
- Header guard: `PROJECT_PATH_FILE_H_`.
- Prefer `#include` over forward declares when practical (tooling/dependency clarity).
- Short functions may be in headers; large bodies in `.cc`. No `-inl.h` pattern for new code.

## Scoping / globals

- Code in **namespaces** named from project/path; **no** `using namespace foo` in headers (avoid in most .cc too).
- Static/global objects must be **trivially destructible** unless carefully justified; prefer `constexpr` when possible.

## Classes

- `struct` = passive data; `class` = invariants/behavior.
- Data members **private** (except constants); composition > inheritance; public inheritance only.
- Prefer single inheritance; multiple **implementation** inheritance discouraged.
- Declaration order: `public` → `protected` → `private`; group types, factories, ctors, dtor, methods, data.
- Operator overloading judicious; **no user-defined literals** for normal code.

## C++ function and type mechanics

- General function/module boundaries are defined in `../design.md`; this file only adds C++ mechanics.
- Inputs before outputs in parameter lists when clear.
- Integers: plain `int` or `<cstdint>` exact widths (`int64_t`…); care with intermediate overflow.
- Prefer **preincrement** `++i` when value unused.
- APIs: `const`/`constexpr` aggressively when correct.
- Casts: **C++ casts** (`static_cast`, etc.) or brace conversion; avoid C casts except to `void`.
- Avoid **RTTI** (`dynamic_cast`/`typeid`) except tests/interop corners.
- Exceptions: follow project policy (Google historically restricted; match codebase).

## Naming (shape encodes kind)

| Kind | Form |
|---|---|
| type | `PascalCase` / `CamelCase` types |
| variable / function | `snake_case` |
| class data member | `snake_case_` trailing underscore |
| constant | `kCamelCase` (Google) or project-local variant — **match file** |
| enum | type `PascalCase`; enumerators per guide / local |
| macro | `ALL_CAPS` — avoid macros when possible |
| namespace | `snake_case` |

## Comments / format

- File/class/function comments for non-obvious contracts; prefer self-explanatory names.
- Formatting owned by clang-format Google style — don’t bikeshed after format.

## Agent anti-patterns

```cpp
using namespace std;          // No in headers
static std::string g_path;   // non-trivial dtor global — avoid
(int)x;                      // prefer static_cast<int>(x)
```
