# Python — Google Style (delta)

Upstream: `pyguide.md`. Prefer project `ruff`/`black` if present; Google historically ~**80** cols (exceptions: long imports, URLs, pylint pragmas).

## Imports

- Follow the project's established import shape first; package topology and public API design determine whether direct or module-qualified imports are clearer.
- With no local precedent, direct imports can keep type annotations readable, while module-qualified calls can preserve useful origin context for helper-heavy APIs. Treat this as a tradeoff, not a universal split between types and functions.
- `from x import y as z` **only** for: name clash, public API parameter clash, inconveniently long name, or overly generic `y`.
- **Never** `from x import y as y`.
- Relative imports are not inherently wrong; follow the package's established public/internal boundary and tooling.
- Order (common): future → stdlib → third party → first party; one blank line between groups. Exact sort: follow `isort` profile if configured.

## Language bans / sharp edges

| Rule | Do |
|---|---|
| Mutable defaults | `def f(x=None):` then `if x is None: x = []` — never `def f(x=[])` |
| Default arg timing | Don’t use `time.time()` / flag values evaluated at **def** time |
| `assert` | Not for required runtime validation (stripped with `-O`) — raise |
| Bare `except:` / broad `except Exception` | Avoid unless re-raise or true isolation boundary |
| Mutable module globals | Avoid; constants `ALL_CAPS`; mutate via functions if must |
| Power features | Avoid gratuitous metaclass / heavy introspection / clever `exec` |
| Implicit truth | Prefer `if not users:` over `len(users)==0`; **always** `is None` for None; careful with ints (`if not i % 10` is a footgun) |
| Lambdas | Prefer def for non-trivial; no assignment of complex lambda |
| Nested classes/functions | OK when justified; don’t nest for theater |

## Naming

| Kind | Form |
|---|---|
| module/file | `lower_with_under.py` — **no dashes** |
| class | `CapWords` |
| function/method/var | `lower_with_under` |
| constant | `ALL_CAPS` |
| internal | single leading `_` preferred over `__` name-mangle for privacy |
| avoid | useless type suffixes (`name_string`), offensive names, random 1-letter beyond `i/j/k/e/f` idioms |

## Types and docstrings

- Type annotations encouraged on public APIs; use modern `list[int]`, `X | None` when runtime allows.
- Docstrings: public modules/functions/classes/methods; summary line; args/returns when non-obvious.

## Python-specific type mechanics

Structural selection between functions, objects, records, and polymorphism is defined only in `../design.md`.

- `@classmethod` must actually use `cls`; it commonly implements alternate constructors or type-level policy.
- `@staticmethod` is appropriate only when an operation belongs to an already meaningful type API while using neither instance nor type.
- Use `@dataclass` for record-like data instead of hand-writing boilerplate without additional behavior.
- Define an inner class only when Python scope, lifecycle, state, or protocol semantics require it.

```python
# One acceptable pattern when the project has no stronger convention:
# direct imports keep signature types concise.
from billing.models import Invoice
from billing.types import CurrencyCode

# Module qualification preserves useful ownership at helper call sites.
from billing import pricing

amount = pricing.amount_for(invoice)
```

## Yes / No

```python
# Yes
def connect(min_port: int = 1024) -> int:
    if min_port < 1024:
        raise ValueError(...)
    ...

# No
def connect(min_port: int = 1024) -> int:
    assert min_port >= 1024
    ...

# Yes
def append(item, bucket=None):
    if bucket is None:
        bucket = []
    bucket.append(item)
    return bucket

# No
def append(item, bucket=[]):
    bucket.append(item)
    return bucket
```
