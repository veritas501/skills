# AI failure-pattern examples

Use after diagnosing Class Theater, Fake Modularity, useless aliases, or Speculative Generality in `../refactoring.md`.

## Useless import alias (AI smell)

```python
# BAD
from package.module.service import Handler as Handler
from package.module.service import Handler as H  # only OK if H is a deliberate short local name used heavily

# GOOD
from package.module.service import Handler
```

## Fake encapsulation by file-split only

```python
# BAD: three files, same bag of args, no owner
# billing_utils.py
def calc_amount(perf, plays, rules, taxes): ...
def calc_credits(perf, plays, rules): ...
# billing_format.py
def format_line(perf, plays, amount): ...
# billing.py
def statement(invoice, plays, rules, taxes):
    for perf in invoice["performances"]:
        a = calc_amount(perf, plays, rules, taxes)
        c = calc_credits(perf, plays, rules)
        ...

# GOOD: own the data the ops need; thin orchestration
# statement_data.py — build structured data (phase 1)
# statement_plain_text.py — render (phase 2)
# pricing.py — PerformanceCalculator / amount_for next to play rules
```

## Speculative generality

```python
# BAD: one implementation, full plugin architecture
class AbstractExporter(ABC):
    @abstractmethod
    def export(self, data): ...
class PluginRegistry:
    def register(self, name, exporter): ...
class JsonExporter(AbstractExporter):
    def export(self, data): return json.dumps(data)
registry = PluginRegistry(); registry.register("json", JsonExporter())

# GOOD: you have one format today
def export_json(data):
    return json.dumps(data)
# When a second format appears, then extract a shared shape.
```
