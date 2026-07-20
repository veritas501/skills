# Theater statement: complete Split Phase example

This example demonstrates `../../recipes/split-phase.md` and `../../recipes/replace-conditional-with-polymorphism.md`.

## Ownership

```text
statement.js
  public entrypoints + plain-text/HTML rendering only

createStatementData.js
  source lookup + enrichment + totals + variant-specific pricing/credits
```

## Data handoff

```js
{
  customer,
  performances: [{ playID, audience, play, amount, volumeCredits }],
  totalAmount,
  totalVolumeCredits
}
```

Renderers consume this structure only. They never reopen `plays` or repeat pricing rules.

## Dependency direction

```text
statement(invoice, plays)
  -> createStatementData(invoice, plays)
  -> renderPlainText(data) / renderHtml(data)
```

- `statement.js` depends on the data builder.
- The data builder does not depend on rendering.
- Variant selection is owned once by the calculator factory.
- Amount and credit behavior live on the variant calculators.

## Source files

- `createStatementData.js`: the single source of truth for the data phase.
- `statement.js`: the single source of truth for rendering.

Expected sample behavior: BigCo owes `$1,730.00` and earns `47` credits for the chapter fixture.
