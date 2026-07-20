# Fowler core examples

Demonstrates moves from `../refactoring.md`: extraction, dead-parameter removal, Split Phase, polymorphism, parameter objects, and guard clauses. Examples show shape; the refactoring/recipe files remain authoritative.

## Extract Function (Fowler ch6)

Name by **what**, not how. Even a one-liner is fine if the name carries intent.

```js
// BAD: long function with comment-as-section-header
function printOwing(invoice) {
  let outstanding = 0;
  console.log("***********************");
  console.log("**** Customer Owes ****");
  console.log("***********************");
  for (const o of invoice.orders) outstanding += o.amount;
  // print details
  console.log(`name: ${invoice.customer}`);
  console.log(`amount: ${outstanding}`);
}

// GOOD: extract named intentions
function printOwing(invoice) {
  printBanner();
  const outstanding = calculateOutstanding(invoice);
  printDetails(invoice, outstanding);
}
function printBanner() {
  console.log("***********************");
  console.log("**** Customer Owes ****");
  console.log("***********************");
}
function printDetails(invoice, outstanding) {
  console.log(`name: ${invoice.customer}`);
  console.log(`amount: ${outstanding}`);
}
function calculateOutstanding(invoice) {
  let result = 0;
  for (const o of invoice.orders) result += o.amount;
  return result;
}
```

## Remove a truly dead parameter

Remove a parameter only after proving it no longer participates in behavior. Do not replace an explicit dependency with a closure, global, service locator, or hidden lookup.

```js
// BAD: play is no longer used after pricing moved onto the calculator.
function amountFor(calculator, play) {
  return calculator.amount;
}
const amount = amountFor(calculator, calculator.play);

// GOOD: remove only the proven-unused parameter.
function amountFor(calculator) {
  return calculator.amount;
}
const amount = amountFor(calculator);
```

## Split Phase (Fowler ch1 / ch6)

Separate **calculation** from **formatting**. Do not grow one statement function forever.

```js
// BAD: one function computes totals AND builds the string
function statement(invoice, plays) {
  let totalAmount = 0, volumeCredits = 0, result = `Statement for ${invoice.customer}\n`;
  for (let perf of invoice.performances) {
    // pricing + credits + string append mixed
  }
  result += `Amount owed is ${format(totalAmount/100)}\n`;
  return result;
}

// GOOD: data phase, then render phase
function statement(invoice, plays) {
  return renderPlainText(createStatementData(invoice, plays));
}
// createStatementData → { customer, performances[{play,amount,volumeCredits}], totals }
// renderPlainText(data) → strings only; never reopens plays / pricing
```

**Full ch1 end-state** (two modules + polymorphic calculators):  
`theater-statement/overview.md`. Use `../recipes/split-phase.md` as the authoritative procedure.

## Polymorphism over repeated type switch (Fowler ch1 / ch10)

Own variant logic **once**.

```js
// BAD: switch(play.type) copied for amount, credits, ...
function amountFor(perf) {
  switch (playFor(perf).type) {
    case "tragedy": /* ... */ break;
    case "comedy": /* ... */ break;
  }
}

// GOOD: calculator hierarchy (or one dispatch table) owns the variants
class PerformanceCalculator {
  constructor(aPerformance, aPlay) {
    this.performance = aPerformance;
    this.play = aPlay;
  }
  get amount() { throw new Error("subclass responsibility"); }
  get volumeCredits() {
    return Math.max(this.performance.audience - 30, 0);
  }
}
class TragedyCalculator extends PerformanceCalculator {
  get amount() { /* tragedy rules */ }
}
class ComedyCalculator extends PerformanceCalculator {
  get amount() { /* comedy rules */ }
  get volumeCredits() {
    return super.volumeCredits + Math.floor(this.performance.audience / 5);
  }
}
function createPerformanceCalculator(aPerformance, aPlay) {
  switch (aPlay.type) {
    case "tragedy": return new TragedyCalculator(aPerformance, aPlay);
    case "comedy": return new ComedyCalculator(aPerformance, aPlay);
    default: throw new Error(`unknown type: ${aPlay.type}`);
  }
}
```

## Introduce Parameter Object (Fowler ch6)

```js
// BAD: data clump in every signature
function withinRange(aReading, min, max) {
  return aReading.temp >= min && aReading.temp <= max;
}
const station = { name: "ZB1", readings: [/* ... */] };
const operatingPlan = { temperatureFloor: 50, temperatureCeiling: 60 };
alerts = station.readings.filter(r => !withinRange(r, operatingPlan.temperatureFloor, operatingPlan.temperatureCeiling));

// GOOD
class NumberRange {
  constructor(min, max) { this._data = { min, max }; }
  get min() { return this._data.min; }
  get max() { return this._data.max; }
  contains(arg) { return arg >= this.min && arg <= this.max; }
}
function withinRange(aReading, range) {
  return range.contains(aReading.temp);
}
const range = new NumberRange(operatingPlan.temperatureFloor, operatingPlan.temperatureCeiling);
alerts = station.readings.filter(r => !withinRange(r, range));
```

## Guard clauses vs nested conditionals (Fowler ch10)

```js
// BAD
function payAmount(employee) {
  let result;
  if (employee.isSeparated) {
    result = { amount: 0, reasonCode: "SEP" };
  } else {
    if (employee.isRetired) {
      result = { amount: 0, reasonCode: "RET" };
    } else {
      // logic to compute amount
      result = someFinalComputation();
    }
  }
  return result;
}

// GOOD
function payAmount(employee) {
  if (employee.isSeparated) return { amount: 0, reasonCode: "SEP" };
  if (employee.isRetired) return { amount: 0, reasonCode: "RET" };
  return someFinalComputation();
}
```
