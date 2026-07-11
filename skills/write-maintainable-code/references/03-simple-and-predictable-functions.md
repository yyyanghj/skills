# Simple and Predictable Functions

A function should do one coherent thing. Prefer small functions with one clear responsibility, but accept a longer linear function when the work itself is one cohesive unit. Do not split code merely to reduce line count.

Make inputs, outputs, state, and side effects obvious. Prefer the same input to produce the same result unless the function clearly exists to perform an effect.

## Prefer

- Small functions with one clear responsibility.
- Pure functions for business rules and data transformations.
- Explicit parameters and return values instead of hidden shared state.
- New values by default; keep mutation local and explicit when it is simpler.
- Long linear functions when the logic is one coherent unit.
- A helper for a stable sub-responsibility, behavior repeated across current callers, or separately meaningful policy.
- A helper or seam that isolates an unstable dependency such as an SDK, clock, filesystem, or network client.
- Collection helpers only when they are clearer than a direct loop.

## Avoid

- Mixing calculation with unrelated logging, analytics, persistence, or UI effects.
- Reading mutable module globals or singletons implicitly.
- Mutating inputs without an explicit API contract.
- Functions that combine unrelated responsibilities only because they run in sequence.
- Splitting a coherent function solely to reduce its length.
- Helpers that only rename, forward, or hide one local expression.
- Extracting a helper before its responsibility or reuse pattern is clear.

## Design and Review Check

Use these questions while designing functions and reviewing the implementation:

- Can the function's coherent responsibility be described in one sentence?
- Are its inputs, outputs, state, and effects explicit?
- If the function is long, is the logic still linear and one cohesive unit?
- If a helper is proposed, what stable responsibility, repeated behavior, policy, or boundary does it own?
- Would inlining the helper make the parent function easier to understand?
- Can business calculations be tested without infrastructure?

## Examples

### Example: Extract a Meaningful Calculation

Avoid mixing a reusable calculation with an unrelated effect:

```ts
function priceOrder(order: Order, taxRate: number): Order {
  const subtotal = order.items.reduce((sum, item) => sum + item.price, 0);
  const pricedOrder = {
    ...order,
    subtotal,
    total: subtotal * (1 + taxRate),
  };

  analytics.track("order_priced", { orderId: order.id });
  return pricedOrder;
}
```

Prefer extracting the coherent calculation and keeping the effect visible at the call site:

```ts
function calculateOrderPrice(order: Order, taxRate: number): Order {
  const subtotal = order.items.reduce((sum, item) => sum + item.price, 0);

  return {
    ...order,
    subtotal,
    total: subtotal * (1 + taxRate),
  };
}

const pricedOrder = calculateOrderPrice(order, taxRate);
analytics.track("order_priced", { orderId: order.id });
```

### Example: Keep a Coherent Transformation Together

Avoid one-use helpers that fragment a simple transformation:

```ts
function normalizeName(input: UserInput): string {
  return input.name.trim();
}

function normalizeEmail(input: UserInput): string {
  return input.email.trim().toLowerCase();
}

function normalizeUser(input: UserInput): User {
  return {
    name: normalizeName(input),
    email: normalizeEmail(input),
  };
}
```

Prefer the longer but still linear and cohesive function:

```ts
function normalizeUser(input: UserInput): User {
  const name = input.name.trim();
  const email = input.email.trim().toLowerCase();

  return { name, email };
}
```
