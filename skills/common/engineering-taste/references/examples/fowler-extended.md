# Fowler extended examples

Demonstrates ownership moves, query extraction, encapsulation, API separation, and conditional restructuring. Rules live in `../design.md` and `../refactoring.md`.

## Feature Envy → move next to data (Fowler ch7/ch8)

```js
// BAD: function lives outside, picks another object's pockets
function getRating(driver) {
  return moreThanFiveLateDeliveries(driver) ? 2 : 1;
}
function moreThanFiveLateDeliveries(driver) {
  return driver.numberOfLateDeliveries > 5;
}

// GOOD
class Driver {
  get rating() {
    return this.moreThanFiveLateDeliveries ? 2 : 1;
  }
  get moreThanFiveLateDeliveries() {
    return this.numberOfLateDeliveries > 5;
  }
}
```

## Replace Temp with Query (Fowler ch7)

```js
// BAD: temps block extraction; price logic trapped in method body
get price() {
  const basePrice = this._quantity * this._itemPrice;
  const discountFactor = basePrice > 1000 ? 0.95 : 0.98;
  return basePrice * discountFactor;
}

// GOOD
get price() {
  return this.basePrice * this.discountFactor;
}
get basePrice() { return this._quantity * this._itemPrice; }
get discountFactor() { return this.basePrice > 1000 ? 0.95 : 0.98; }
```

## Extract Function with local vars as params (ch6)

```js
// BAD
function printOwing(invoice) {
  let outstanding = calculateOutstanding(invoice);
  // record due date
  const today = Clock.today;
  invoice.dueDate = new Date(today.getFullYear(), today.getMonth(), today.getDate() + 30);
  // print details
  console.log(`name: ${invoice.customer}`);
  console.log(`amount: ${outstanding}`);
  console.log(`due: ${invoice.dueDate.toLocaleDateString()}`);
}

// GOOD
function printOwing(invoice) {
  const outstanding = calculateOutstanding(invoice);
  recordDueDate(invoice);
  printDetails(invoice, outstanding);
}
function recordDueDate(invoice) {
  const today = Clock.today;
  invoice.dueDate = new Date(today.getFullYear(), today.getMonth(), today.getDate() + 30);
}
function printDetails(invoice, outstanding) {
  console.log(`name: ${invoice.customer}`);
  console.log(`amount: ${outstanding}`);
  console.log(`due: ${invoice.dueDate.toLocaleDateString()}`);
}
```

## Inline Function when name adds nothing (ch6 reverse)

```js
// BAD: this wrapper repeats the field access and carries no domain meaning.
function customerName(customer) {
  return customer.name;
}
const heading = `Customer: ${customerName(customer)}`;

// GOOD
const heading = `Customer: ${customer.name}`;
```

Do not inline a named domain predicate merely because its body is short. A name such as `isEligibleForRenewal` can preserve business meaning even when implemented by one comparison.

## Combine Functions into Class (ch6)

Use a class when several behaviors share the same data and jointly express one coherent policy or invariant. If the functions are independent calculations, keep them free.

```js
// BAD: the same reading data and charge policy travel through every function.
function baseCharge(reading) {
  return baseRate(reading.month, reading.year) * reading.quantity;
}
function taxableCharge(reading) {
  return Math.max(0, baseCharge(reading) - taxThreshold(reading.customer));
}

// GOOD: Reading owns the data and the coherent charge policy built on it.
class Reading {
  constructor(data) {
    this._customer = data.customer;
    this._quantity = data.quantity;
    this._month = data.month;
    this._year = data.year;
  }
  get baseCharge() {
    return baseRate(this._month, this._year) * this._quantity;
  }
  get taxableCharge() {
    return Math.max(0, this.baseCharge - taxThreshold(this._customer));
  }
}
```

## Encapsulate Record (ch7)

```js
// BAD: raw record can enter invalid states from any caller.
const organization = { name: "Acme Gooseberries", country: "GB" };
result += `<h1>${organization.name}</h1>`;
organization.name = "";
organization.country = "United Kingdom";

// GOOD: the object owns real invariants and controlled updates.
class Organization {
  constructor(data) {
    this.rename(data.name);
    this.changeCountry(data.country);
  }
  get name() { return this._name; }
  get country() { return this._country; }
  rename(name) {
    if (!name.trim()) throw new Error("organization name must not be empty");
    this._name = name;
  }
  changeCountry(code) {
    if (!/^[A-Z]{2}$/.test(code)) throw new Error("country must be an ISO alpha-2 code");
    this._country = code;
  }
}
const organization = new Organization({ name: "Acme Gooseberries", country: "GB" });
```

If the record has no invariant, policy, or controlled-update need, keep it as a plain immutable record instead of manufacturing a class.

## Replace Primitive with Object (ch7 priority)

```js
// BAD: stringly priority compared ad hoc
if (order.priority === "high" || order.priority === "rush") ...

// GOOD
class Priority {
  constructor(value) { this._value = value; }
  higherThan(other) { /* ordered ranks */ }
  toString() { return this._value; }
}
// order stores Priority; comparisons go through methods
```

## Decompose Conditional (ch10)

```js
// BAD
if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
  charge = quantity * plan.summerRate;
else
  charge = quantity * plan.regularRate + plan.regularServiceCharge;

// GOOD
if (summer()) charge = summerCharge();
else charge = regularCharge();

function summer() {
  return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd);
}
function summerCharge() { return quantity * plan.summerRate; }
function regularCharge() {
  return quantity * plan.regularRate + plan.regularServiceCharge;
}
```

## Separate Query from Modifier (ch11)

```js
// BAD: looking up a miscreant also alerts — surprise side effect
function getTotalOutstandingAndSendBill() {
  const result = customer.invoices.reduce((total, each) => each.amount + total, 0);
  sendBill();
  return result;
}

// GOOD
function getTotalOutstanding() {
  return customer.invoices.reduce((total, each) => each.amount + total, 0);
}
function sendBill() { /* side effect only */ }
```

## Remove Flag Argument (ch11)

```js
// BAD
function bookConcert(aCustomer, isPremium) {
  if (isPremium) { /* premium path */ }
  else { /* regular path */ }
}
// calls: bookConcert(c, true) / bookConcert(c, false) — boolean is a cryptic flag

// GOOD
function premiumBookConcert(aCustomer) { /* ... */ }
function regularBookConcert(aCustomer) { /* ... */ }
// or two clearly named entry points that share private helpers
```

## Slide Statements then Extract (ch8)

Group related lines before extract so the new function is cohesive.

```js
// BAD: mixed concerns in loop body
for (const p of people) {
  result += `<td>${p.name}</td>`;
  // later unrelated
  if (p.age > 18) adults += 1;
  totalAge += p.age;
}

// GOOD: slide, then extract
for (const p of people) {
  result += renderPersonCell(p);
}
// age aggregation elsewhere or second loop (Split Loop if needed)
```
