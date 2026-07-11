# Flat and Linear Control Flow

Keep execution flat, linear, and local. A reader should be able to trace the path from entry point to effect from top to bottom without repeatedly jumping across branches, callbacks, helpers, or files.

Flat control flow does not mean placing everything in one function. Keep meaningful boundaries, but do not create one-off helpers whose only effect is forcing the reader to jump elsewhere for a local detail.

## Prefer

- Guard clauses for invalid state and edge cases.
- One visible orchestrator that calls meaningful operations in execution order.
- Direct local variables, loops, and branches.
- Short one-use expressions kept near the code that needs them.
- State changes close to the conditions that trigger them.
- Helpers that own real logic, policy, stable behavior repeated across current callers, or an unstable dependency boundary.

## Avoid

- Deeply nested `if`, `else`, or callback chains.
- One-off helpers that only rename a value, access one property, or forward one call.
- Splitting a local flow into many single-call steps.
- Callback parameters whose only purpose is to continue synchronous execution elsewhere.
- Dispatcher chains that forward to another dispatcher.
- Forcing readers to bounce across files to reconstruct one execution path.
- Hiding important branches behind vague helper names.

## Design and Review Check

Use these questions while designing the flow and reviewing the implementation:

- Can a reader follow the main path from top to bottom?
- Is the happy path visually dominant?
- Are important branches and state changes visible near their effects?
- Does each helper own meaningful logic, policy, stable behavior repeated across current callers, or a real boundary?
- Would inlining a one-use helper make the code easier to trace?
- Does the design introduce callbacks or files that only continue the same synchronous flow?

## Examples

### Example: Flatten Branches

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

Prefer guard clauses with the same failure behavior:

```ts
function saveUser(user: User | null): void {
  if (!user) throw new Error("User is required");
  if (!user.isActive) throw new Error("User is inactive");
  if (!user.canEdit) throw new Error("User cannot edit");

  persistUser(user);
}
```

### Example: One-Use Helper

Avoid a helper that only hides a local property access:

```ts
function getReceiptEmail(order: Order): string {
  return order.customer.email;
}

function completeCheckout(order: Order): void {
  persistOrder(order);
  sendReceipt(getReceiptEmail(order), order.id);
}
```

Prefer keeping the detail at its single point of use:

```ts
function completeCheckout(order: Order): void {
  persistOrder(order);
  sendReceipt(order.customer.email, order.id);
}
```

### Example: Traceable Execution Path

Avoid callback wrappers that make one synchronous checkout flow jump between files.

`checkout.ts`:

```ts
import { chargePayment, loadCart, reserveInventory } from "./checkout-steps.js";

loadCart((cart) => {
  const order = startCheckout(cart);
  validateOrder(order);

  reserveInventory(order, () => {
    chargePayment(calculateTotal(order));
    sendReceipt(order);
    finishCheckout(order);
  });
});
```

`checkout-steps.ts`:

```ts
import { readCart, writeInventoryReservation } from "./checkout-store.js";
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

Prefer one visible orchestrator that calls the real boundaries directly:

```ts
import { readCart, writeInventoryReservation } from "./checkout-store.js";
import { submitPayment } from "./payment-gateway.js";

function runCheckout(): void {
  const cart = readCart();
  const order = startCheckout(cart);
  validateOrder(order);
  writeInventoryReservation(order);
  submitPayment(calculateTotal(order));
  sendReceipt(order);
  finishCheckout(order);
}

runCheckout();
```

### Example: Nested Ternary

Given:

```ts
interface ShippingOrder {
  isInternational: boolean;
  totalInCents: number;
}
```

Avoid compressing several business rules into a nested expression:

```ts
function calculateShippingFeeInCents(order: ShippingOrder): number {
  return order.isInternational
    ? order.totalInCents >= 10_000
      ? 0
      : 2_500
    : order.totalInCents >= 5_000
      ? 0
      : 500;
}
```

Prefer explicit branches whose decision order is immediately visible:

```ts
function calculateShippingFeeInCents(order: ShippingOrder): number {
  if (order.isInternational && order.totalInCents >= 10_000) return 0;
  if (order.isInternational) return 2_500;
  if (order.totalInCents >= 5_000) return 0;
  return 500;
}
```
