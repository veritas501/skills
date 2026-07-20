# Go — Google Style (delta)

Upstream: `go/guide.md` (canonical principles), `go/decisions.md`, `go/best-practices.md`.

## Non-negotiable tooling

- Run **`gofmt`** (or `goimports`) on every change. Do not hand-format against it.
- **MixedCaps** (not snake_case) for exported and unexported names; unexported starts lowercase.
- No forced line-length religion; wrap for **clarity**, not an 80-col cult.

## Principles (priority)

1. **Clarity** (reader > writer)  
2. **Simplicity**  
3. **Concision** (signal/noise)  
4. **Maintainability**  
5. **Consistency** with local codebase  

Comments: explain **why** and non-obvious constraints; don’t restate the code.

## Naming (high-churn decisions)

| Topic | Pin |
|---|---|
| Package names | short, lower case, **no underscores**, no `util`/`common` grab-bags |
| Initialisms | Keep consistent case: `URL`/`HTTP` as units (`ServeHTTP`, `urlPath` unexported) |
| Getters | no `Get` prefix: field `name` → method `Name()` |
| Receiver | short (1–3 letters), consistent for type; not `this`/`self` |
| Repetition | drop stutter (`package book`; type `Book` not `book.BookBook`) |
| Error vars | `ErrX` / `errX`; type often `XError` |

## Errors

- Return `error` as **last** result; handle immediately; **don’t** discard with `_` without reason.
- Error strings: **lowercase**, no trailing punctuation (not like English sentences).
- Prefer `fmt.Errorf("...: %w", err)` for wrap; place `%w` so chain stays inspectable.
- Indent happy path left; **guard clauses** for errors (`if err != nil { return ..., err }`).
- **Don’t panic** in libraries for ordinary failure; `Must*` only for irreversible prog init (and document).
- No in-band error sentinels in values when `error` works.

## Language / API

- Pass **small structs by value** unless mutation/identity/large size needs pointer (see decisions “Pass values” / receiver type).
- Avoid **unnecessary interfaces**; define interfaces on the **consumer** side when practical.
- Prefer synchronous APIs; document goroutine lifetime if you start one (must be stoppable/accounted).
- Nil slices: prefer `var s []T` / nil over empty literal unless JSON/non-nil semantics needed.
- Context: first param `ctx context.Context`; don’t store ctx in structs except rare request-scoped objects.

## Tests

- Table-driven tests common; use `t.Run`.
- `t.Fatal` vs `t.Error`: fatal when continuation is nonsense.
- Useful failures: identify function + input; **got before want**; stable diffs.
- Prefer package `testing`; avoid assertion libraries that hide diffs (Google decision).

## Imports

- Group with blank lines (std / external / local) per `goimports`.
- Avoid `import .` ; blank imports only for side-effect registration and comment why.
