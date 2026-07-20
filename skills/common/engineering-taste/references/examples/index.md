# Example routing

Load the smallest example file matching the diagnosed problem. Examples demonstrate authoritative rules; they never define new ones.

| Problem | Load | Authoritative rule |
|---|---|---|
| Long function, extraction, parameter object, guard clauses | `fowler-core.md` | `../refactoring.md` |
| Feature Envy, encapsulation, condition/API restructuring | `fowler-extended.md` | `../design.md` and `../refactoring.md` |
| Useless alias, fake file split, speculative framework | `ai-failures.md` | `../refactoring.md` |
| Python function/class/classmethod/staticmethod/import boundaries | `python-boundaries.md` | `../design.md` + `../style/python.md` |
| Complete calculation/render Split Phase | `theater-statement/overview.md` | `../recipes/split-phase.md` |

Do not load every example file by default. Diagnose first, then select one nearest shape.
