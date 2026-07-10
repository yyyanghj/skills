# Keep Functions Simple and Predictable

Make each function's inputs, outputs, state, and side effects obvious. Prefer the same input to produce the same result unless the function clearly exists to perform an effect.

## Prefer

- Pure functions for business rules and data transforms.
- Explicit parameters and return values instead of hidden shared state.
- New values by default; keep mutation local and explicit when it is simpler.
- Long linear functions when the logic is one coherent unit.
- Collection helpers only when they are clearer than a loop.

## Avoid

- Mixing calculation with unrelated logging, analytics, persistence, or UI effects.
- Reading mutable module globals or singletons implicitly.
- Mutating inputs without an explicit API contract.
- Functions that mix validation, orchestration, persistence, and formatting without a coherent boundary.
- Splitting functions solely to reduce line count.

Extract a helper when a real sub-responsibility, repeated behavior, reuse site, or useful test seam appears. Reuse or testability alone does not justify pure forwarding; a thin seam must isolate a meaningful unstable dependency such as an SDK, clock, or filesystem under Core Avoid.

## Review Check

- Can the function be described in one sentence?
- Are inputs, outputs, state, and effects explicit?
- Would reordering calls change hidden shared state?
- Does each extracted helper own meaningful behavior or isolate a meaningful unstable dependency boundary?
- Can the calculation be tested without infrastructure?

## Example: Separate Calculation from Effects

Avoid combining a calculation with an unrelated effect:

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

Keep the calculation pure and let the caller own the same effect:

```ts
function calculateOrderPrice(order: Order, taxRate: number): Order {
  const subtotal = order.items.reduce((sum, item) => sum + item.price, 0);

  return {
    ...order,
    subtotal,
    total: subtotal * (1 + taxRate),
  };
}

function priceOrder(order: Order, taxRate: number): Order {
  const pricedOrder = calculateOrderPrice(order, taxRate);
  analytics.track("order_priced", { orderId: order.id });
  return pricedOrder;
}
```
