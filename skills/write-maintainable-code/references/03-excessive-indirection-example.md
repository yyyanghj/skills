# Avoid Excessive Indirection

The following callback wrappers make one synchronous checkout flow jump between two files.

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

Both versions run the same operations in the same order. The second removes continuation callbacks and forwarding wrappers, so the execution order is visible without cross-file call-stack reconstruction.
