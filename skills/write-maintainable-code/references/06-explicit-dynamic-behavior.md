# Explicit Dynamic Behavior

Keep behavior selection, dependency wiring, and runtime effects visible. A maintainer should understand how code is chosen and invoked without relying on import side effects, hidden registration, or unusual language knowledge.

Treat portability as a tiebreaker, not a ban. When alternatives are equally clear, prefer familiar constructs with explicit semantics that transfer across languages. Use a language-specific or advanced feature when it materially reduces complexity, fits the repository, and remains understandable in its local context.

## Prefer

- Direct calls and visible dependency wiring.
- Declarative configuration with explicit execution.
- Plain data, functions, enums or tagged unions, branches, and loops when they express the behavior clearly.
- Validation where untyped or external data enters the system.
- Explicit rejection of unknown cases instead of undefined behavior.
- Exhaustive branches for closed sets of variants.
- Explicit `switch` or `if` statements for behavior dispatch.
- Lookup tables for static data mappings, not action, rule, or strategy dispatch.

## Avoid

- Registration through import side effects.
- String-based dispatch without input validation.
- Action, rule, or strategy maps that hide behavior behind dynamic lookup.
- Decorators or metaprogramming that hide ownership or execution order.
- Prototype mutation, metaclasses, or dynamic method synthesis when a direct construct works.
- Runtime-heavy pipelines when a direct branch or loop is clearer.
- Type-level machinery that is harder to understand than the data it describes.
- Advanced features used for novelty rather than lower total cognitive load.

## Design and Review Check

Use these questions while designing dynamic behavior and reviewing the implementation:

- Can a reader see how behavior is selected, wired, and invoked?
- Is dynamic input validated before dispatch?
- Do unknown cases fail explicitly and diagnostically?
- Is every closed variant handled exhaustively?
- Does the advanced feature reduce more complexity than it adds?
- When alternatives are equally clear, does the design use the more familiar and portable construct?

## Examples

### Example: Closed Variants

Avoid behavior scattered through an inheritance hierarchy:

```ts
abstract class Shape {
  abstract area(): number;
}

class Circle extends Shape {
  constructor(readonly radius: number) {
    super();
  }

  area(): number {
    return Math.PI * this.radius ** 2;
  }
}

class Square extends Shape {
  constructor(readonly side: number) {
    super();
  }

  area(): number {
    return this.side ** 2;
  }
}
```

Prefer explicit variants, centralized dispatch, and an exhaustive failure path:

```ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function assertNever(value: never): never {
  throw new Error(`Unsupported shape: ${JSON.stringify(value)}`);
}

function calculateArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    default:
      return assertNever(shape);
  }
}
```

### Example: Explicit Behavior Dispatch

Given a closed set of payment variants:

```ts
type PaymentKind = "card" | "paypal";

interface Payment {
  kind: PaymentKind;
  amountInCents: number;
}
```

Avoid hiding action dispatch behind a strategy map:

```ts
type PaymentHandler = (payment: Payment) => Promise<void>;

const paymentHandlers: Record<Payment["kind"], PaymentHandler> = {
  card: handleCard,
  paypal: handlePayPal,
};

async function dispatchPayment(payment: Payment): Promise<void> {
  await paymentHandlers[payment.kind](payment);
}
```

Prefer an explicit and exhaustive branch:

```ts
async function dispatchPayment(payment: Payment): Promise<void> {
  switch (payment.kind) {
    case "card":
      await handleCard(payment);
      return;
    case "paypal":
      await handlePayPal(payment);
      return;
    default:
      throw new Error("Unsupported payment kind");
  }
}
```
