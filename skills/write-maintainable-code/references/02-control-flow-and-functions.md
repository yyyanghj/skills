# Keep Control Flow and Functions Flat and Linear

Keep execution flat, linear, and local. A reader should be able to trace the path from entry point to effect from top to bottom without repeatedly jumping across thin functions, callbacks, or files.

## Contents

- [Rules](#rules)
- [Example: Flatten Branches](#example-flatten-branches)
- [Example: Avoid Excessive Indirection](#example-avoid-excessive-indirection)
- [Example: Separate Calculation from Effects](#example-separate-calculation-from-effects)

## Rules

### Prefer

- Guard clauses for invalid state and edge cases.
- One visible orchestrator that calls leaf operations in execution order.
- Direct local variables, loops, and branches.
- State changes close to the conditions that trigger them.
- Pure functions for calculations and data transforms.
- Explicit parameters and return values instead of hidden shared state.
- New values by default; keep mutation local and explicit when it is simpler.
- Long linear functions when the logic is one coherent unit.

### Avoid

- Deeply nested `if`, `else`, or callback chains.
- Forcing readers to bounce across functions or files to reconstruct one synchronous flow.
- Callback parameters whose only purpose is to continue execution elsewhere.
- Dispatcher chains and helpers that only forward unchanged arguments.
- Extracting microscopic functions solely to reduce line count.
- Mixing calculation with unrelated logging, analytics, persistence, or UI effects.
- Reading mutable module globals or mutating inputs without an explicit contract.
- Collection pipelines that are harder to follow than a loop.

Apply the thin-wrapper threshold from Core Avoid. A domain name, reuse site, test seam, or forwarding call alone is not enough. Split a long flow into a few meaningful stages, not many one-call steps.

### Review Check

- Can a reader follow the main path from top to bottom?
- Are important branches and state changes visible near their effects?
- Does each helper own meaningful logic or policy?
- Would inlining a callback or forwarding layer make execution easier to trace?
- Are effects visible in the function name or at the call site?

## Example: Flatten Branches

Avoid nested control flow:

```ts
function saveUser(user: User | null): void {
  if (user) {
    if (user.isActive) {
      if (user.canEdit) {
        persistUser(user);
      } else {
        throw new Error("User cannot edit");
      }
    } else {
      throw new Error("User is inactive");
    }
  } else {
    throw new Error("User is required");
  }
}
```

Prefer guard clauses with explicit failures:

```ts
function saveUser(user: User | null): void {
  if (!user) throw new Error("User is required");
  if (!user.isActive) throw new Error("User is inactive");
  if (!user.canEdit) throw new Error("User cannot edit");

  persistUser(user);
}
```

## Example: Avoid Excessive Indirection

These callback wrappers make one synchronous checkout flow jump between two files.

`checkout.ts`:

```ts
import { chargePayment, loadCart, reserveInventory } from "./checkout-steps.js";

function validateOrderAndPay(order: Order): void {
  validateOrder(order);

  reserveInventory(order, () => {
    const total = calculateTotal(order);
    chargePayment(total);
    sendReceipt(order);
  });
}

loadCart((cart) => {
  const order = startCheckout(cart);
  validateOrderAndPay(order);
  finishCheckout(order);
});
```

`checkout-steps.ts`:

```ts
import { readCart, writeInventoryReservation} from "./checkout-store.js";
import { submitPayment } from "./payment-gateway.js";

export function loadCart(next: (cart: Cart) => void): void {
  next(readCart());
}

export function reserveInventory(order: Order, next: () => void): void {
  writeInventoryReservation(order);
  next();
}

export function chargePayment(total: number): void {
  submitPayment(total);
}
```

Remove `checkout-steps.ts`. Keep orchestration in one place and call the real I/O boundaries directly.

`checkout.ts`:

```ts
import { readCart, writeInventoryReservation } from "./checkout-store.js";
import { submitPayment } from "./payment-gateway.js";

function runCheckout(): void {
  const cart = readCart();
  const order = startCheckout(cart);
  validateOrder(order);
  writeInventoryReservation(order);
  const total = calculateTotal(order);
  submitPayment(total);
  sendReceipt(order);
  finishCheckout(order);
}

runCheckout();
```

Both versions run the same operations in the same order. The second removes both continuation callbacks and forwarding wrappers, so the execution order is visible without cross-file call-stack reconstruction.

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
