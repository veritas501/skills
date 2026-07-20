# Java — Google Style (delta)

Upstream: `javaguide.html`. Prefer **`google-java-format`** when available.

## File structure

1. License (if any)  
2. `package`  
3. imports  
4. **exactly one** top-level class  

One blank line between sections. One top-level class per file.

## Imports

- **No wildcard imports** (static or otherwise).
- No line-wrap on import lines; column limit doesn’t apply to them.
- Order: static imports group, blank line, non-static group; ASCII sort within group.

## Formatting

- Column limit **100**.
- Braces **K&R**; **always** braces for `if/else/for/do/while` even one-liners.
- 2-space indent (google-java-format).

## Naming

| Kind | Form |
|---|---|
| package | `com.example.deepspace` lowercase, no underscores |
| class/interface | `UpperCamelCase` |
| method/var | `lowerCamelCase` |
| constant | `UPPER_SNAKE_CASE` — only if **deeply immutable** + no side-effect methods |
| test class | ends with `Test` |

## Practices

- **Caught exceptions not ignored** — handle, rethrow, or comment why empty catch is OK.
- Static members: qualify with **class name**, not instance.
- **No `Object.finalize` overrides**.
- Class member order: logical & explainable; overloads stay contiguous.
- Javadoc on public APIs; `/** ... */` form; single-line only when entire block fits and no block tags.

## Agent anti-patterns

```java
import java.util.*; // No — wildcard

try { parse(); } catch (Exception e) {} // No — empty without reason

fooInstance.staticMethod(); // No — use Foo.staticMethod();
```
