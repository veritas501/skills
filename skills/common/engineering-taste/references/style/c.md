# C — no official Google guide (pragmatic delta)

`google/styleguide` has **C++**, not C. When the user asks for “Google style” in C, apply:

1. Project’s own `.clang-format` / Linux kernel / LLVM style if present — **wins**.  
2. Else borrow **shared** discipline from Google C++ where it maps cleanly.  
3. Never invent Google-branded C rules as policy.

## Borrowed pins (C-applicable)

| Topic | Pin |
|---|---|
| Headers | Self-contained; include guards `PROJECT_PATH_FILE_H_`; include what you use |
| Types | Prefer fixed-width `stdint.h` when size matters; plain `int` for general counts |
| Naming | Be consistent: Google C++ uses `snake_case` functions; many C codebases do too — **match neighbors** |
| Globals | Minimize mutable globals; document lifetime/thread rules |
| Macros | Prefer inline functions / enum / static const; `ALL_CAPS` macros if needed |
| Errors | Return error codes/`errno` patterns idiomatic to the codebase; don’t ignore status |
| Format | `clang-format` |

## Explicit non-claims

- Google C++ bans/restrictions on exceptions/RTTI **don’t apply** to C.
- Don’t force C++ `kConstant` naming into a kernel-style C tree.

If writing **C++**, use `cpp.md` instead.
