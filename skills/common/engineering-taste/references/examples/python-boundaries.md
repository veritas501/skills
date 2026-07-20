# Python boundary examples

Demonstrates `../design.md` and `../style/python.md`. These examples do not redefine the rules.

## Stateless class theater (AI smell; collapse)

```python
# BAD: no instance state; still forces construct-then-call
class PriceService:
    def __init__(self):
        pass
    def amount_for(self, performance, play):
        if play["type"] == "tragedy":
            ...
        return result

total = PriceService().amount_for(perf, play)

# STILL BAD: nested class for a pure function
def compute(invoice):
    class _Helper:
        def run(self):
            return sum(x["amount"] for x in invoice["orders"])
    return _Helper().run()

# GOOD: module function
def amount_for(performance, play):
    if play["type"] == "tragedy":
        ...
    return result

# GOOD: alternate constructor actually uses cls
class PerformanceCalculator:
    @classmethod
    def for_performance(cls, performance, play):
        return cls(performance=performance, play=play)
```

## Python module vs class (AI collapse, multi-example)

```python
# BAD A: service class with only config that never varies per instance
class ReportGenerator:
    def __init__(self, fmt="plain"):
        self.fmt = fmt  # always "plain" in entire codebase
    def generate(self, rows):
        return "\n".join(map(str, rows))

# GOOD A
def generate_plain_report(rows):
    return "\n".join(map(str, rows))

# BAD B: classmethod would work; agent uses instance anyway
class PathHelper:
    def join(self, *parts):
        return "/".join(parts)
PathHelper().join("a", "b")

# GOOD B
def join_path(*parts):
    return "/".join(parts)
# or
class PathHelper:
    @staticmethod
    def join(*parts):
        return "/".join(parts)
# Keep this form only if PathHelper is already a meaningful public type/API;
# do not create the class solely to namespace this function.

# BAD C: import noise
from billing.pricing import amount_for as amount_for
from billing import pricing as pricing

# GOOD C
from billing.pricing import amount_for
from billing import pricing  # only if you need the module namespace
```

## Immutable object boundary can still be real

```python
# BAD: the same stable dependencies and invariant travel through every call.
def authorize_order(order, policy, clock): ...
def reserve_order(order, policy, clock): ...
def release_order(order, policy, clock): ...

# GOOD: cohesive capability; no mutable fields required.
class OrderReservation:
    def __init__(self, policy: ReservationPolicy, clock: Clock) -> None:
        self._policy = policy
        self._clock = clock

    def authorize(self, order: Order) -> Authorization: ...
    def reserve(self, order: Order) -> Reservation: ...
    def release(self, order: Order) -> None: ...
```

The class is justified by stable dependencies, shared invariants, and one domain capability—not by mutable state.

## Class method must use the class

```python
# BAD: cls is unused; this is a disguised module function.
class Money:
    @classmethod
    def format(cls, cents: int) -> str:
        return f"${cents / 100:.2f}"

# GOOD: alternate constructor uses cls and remains subclass-aware.
class Money:
    @classmethod
    def from_cents(cls, cents: int) -> "Money":
        return cls(amount=Decimal(cents) / 100)

# GOOD: independent formatting stays at module scope.
def format_usd(cents: int) -> str:
    return f"${cents / 100:.2f}"
```

## Semantic landmarks can beat forced extraction

```python
def deploy_release(release: Release) -> None:
    # Freeze traffic before changing the active artifact. Reversing this order
    # allows requests to observe a partially activated release.
    traffic.freeze(release.service)

    # Activate as one transaction; these calls share rollback state and must
    # remain visibly adjacent.
    snapshot = artifacts.snapshot(release.service)
    artifacts.activate(release)
    health.verify(release.service)

    # Restore service only after verification succeeds.
    traffic.resume(release.service)
```

Do not extract three helpers merely to erase the comments if that hides order, duplicates rollback context, or increases navigation cost. Extract when a phase becomes an independently coherent operation.

## Python imports preserve useful origin context

```python
# Types: direct imports make signatures readable.
from billing.models import Invoice
from billing.types import CurrencyCode

# Operations: module namespace preserves ownership at the call site.
from billing import pricing

def total(invoice: Invoice, currency: CurrencyCode) -> Decimal:
    return pricing.total_for(invoice, currency)
```

Avoid both noisy self-aliases and a flat namespace containing dozens of unrelated helper functions.
