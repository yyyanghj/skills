# Keep Functions Simple and Predictable

Write functions that are easy to trust. Their inputs, outputs, and side effects should be obvious, and the same input should produce the same result unless the function clearly exists to perform an effect.

## Use This Rule When

- Writing business logic.
- Transforming collections or API payloads.
- Refactoring a large function that mixes calculations and side effects.

## Prefer

- Pure functions for business rules and data transforms.
- Returning new objects or arrays instead of mutating shared state.
- Small functions with one clear responsibility.
- Declarative collection helpers when they improve readability.
- **Immutable by default, mutation kept local.** Build data once, then leave it alone; derive new fields or return new values instead of rewriting existing ones.
- **Explicit state passing.** Share state through parameters and return values, not through closures, module-level variables, or singletons.

## Avoid

- Functions that both compute values and trigger unrelated effects.
- Hidden reads from module globals or mutable shared state.
- In-place mutation of inputs unless the API explicitly promises it.
- Long functions that mix validation, orchestration, persistence, and formatting.
- Objects with hidden internal state machines or implicit lifecycle methods.

## Long Functions Are Fine When the Logic Is One Unit

- One function should do **one thing**, but "one thing" is defined by the **logical boundary**, not by line count.
- A complete single pass — parsing one kind of statement, checking one kind of node, handling one request — is usually clearer as one long, linear function than as 20 tiny functions calling each other.
- Split only when a real sub-responsibility, reuse site, or test seam appears. Do not split just to hit a line-count target.

## Practical Heuristics

- Keep a function body small enough to understand in one pass — but length is fine if the whole body is one linear logical flow.
- Separate calculation from I/O whenever possible.
- Make side effects visible in the name or at the call site.
- Prefer `map`, `filter`, and `reduce` only when they are clearer than a loop.

## Review Checklist

- Can you describe the function in one sentence?
- Are the inputs and outputs explicit?
- Would reordering calls change hidden shared state?
- Can the calculation be unit tested without setting up infrastructure?

## Example

**Bad:**

```ts
function applyDiscount(order: Order, taxRate: number) {
  order.subtotal = order.items.reduce((sum, item) => sum + item.price, 0);

  if (order.customer.isVip) {
    order.subtotal = order.subtotal * 0.9;
  }

  order.total = order.subtotal * (1 + taxRate);
  analytics.track("discount_applied", { orderId: order.id });

  return order;
}
```

**Good:**

```ts
function calculateSubtotal(items: OrderItem[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

function calculateTotal(order: Order, taxRate: number): Order {
  const subtotal = calculateSubtotal(order.items);
  const discountedSubtotal = order.customer.isVip ? subtotal * 0.9 : subtotal;

  return {
    ...order,
    subtotal: discountedSubtotal,
    total: discountedSubtotal * (1 + taxRate),
  };
}
```

Keep effects such as analytics, logging, or persistence in a caller that clearly owns them.
