# Simplicity First

The goal of simplicity is to reduce cognitive load. Choose the smallest design that is clearly correct for the current requirement. Optimize for local reasoning: a reader should understand the code in one pass without reconstructing a framework in their head.

Apply this principle while designing architecture, shaping a solution, and preparing a change list, not only while writing code. Avoid over-engineering: do not create abstractions for unknown future needs. Introduce an abstraction when real change pressure reveals a stable concept, or when it enforces a contract or policy, isolates an unstable boundary, or materially simplifies multiple current callers.

## Prefer

- Direct control flow, visible data flow, and small public APIs.
- Straightforward code that is easy to read, debug, and change.
- Concrete types and return shapes that match current callers.
- Existing language and repository patterns before custom frameworks.
- Temporary duplication while the shared concept is still changing.
- Incremental extraction after repeated changes reveal a stable responsibility.
- Focused abstractions for real contracts, policies, or unstable external boundaries.

## Avoid

- Abstract types, factories, registries, or extension points with only speculative uses.
- Service, repository, mapper, or DTO layers without a real boundary or transformation.
- Configuration options that no current caller needs.
- Indirection added only to look scalable or reusable.
- Clever one-liners, compressed control flow, or uncommon language tricks used only to reduce line count.
- Abstractions that remove repeated lines but not repeated changes.
- Optimizations without a measured bottleneck.

## Design and Review Check

Use these questions while designing the solution, preparing the change list, and reviewing the result:

- Can a reader understand the main design in one pass?
- What current requirement or pain justifies each abstraction?
- Does the abstraction represent a stable concept, boundary, contract, or policy?
- Does it simplify multiple current callers or remove repeated changes?
- Would deleting a layer make the happy path clearer without losing real behavior?
- Would a more explicit, boring version be easier to understand and maintain?
- Does the proposed change list include speculative infrastructure unrelated to the current task?

## Examples

### Example: Fixed Validation Rules

Given:

```ts
interface UserInput {
  username: string;
  password: string;
}
```

Avoid a rule engine when two fixed checks are the whole current requirement:

```ts
interface ValidationRule {
  validate(input: UserInput): string | null;
}

class RequiredUsernameRule implements ValidationRule {
  validate(input: UserInput): string | null {
    return input.username.trim() ? null : "Username is required";
  }
}

class MinimumPasswordLengthRule implements ValidationRule {
  validate(input: UserInput): string | null {
    return input.password.length >= 8 ? null : "Password is too short";
  }
}

class ValidationEngine {
  constructor(private readonly rules: ValidationRule[]) {}

  validate(input: UserInput): string[] {
    return this.rules.flatMap((rule) => {
      const error = rule.validate(input);
      return error ? [error] : [];
    });
  }
}

const validator = new ValidationEngine([
  new RequiredUsernameRule(),
  new MinimumPasswordLengthRule(),
]);
```

Prefer the direct implementation until callers need configurable rules:

```ts
function validateUser(input: UserInput): string[] {
  const errors: string[] = [];

  if (!input.username.trim()) errors.push("Username is required");
  if (input.password.length < 8) errors.push("Password is too short");

  return errors;
}
```

### Example: Real External Boundary

Avoid spreading SDK-specific calls and errors through domain logic:

```ts
async function checkout(order: Order): Promise<void> {
  await paymentSdk.pay({ amountInCents: order.totalInCents });
  await saveOrder(order);
}
```

Prefer a focused boundary when it isolates an unstable dependency and owns error translation:

```ts
async function chargePayment(totalInCents: number): Promise<void> {
  try {
    await paymentSdk.pay({ amountInCents: totalInCents });
  } catch (error) {
    throw new Error("Payment failed", { cause: error });
  }
}

async function checkout(order: Order): Promise<void> {
  await chargePayment(order.totalInCents);
  await saveOrder(order);
}
```

### Example: Multiple Array Passes

Simple array methods are fine. Avoid a long chain when it creates intermediate arrays and hides branching or accumulation.

Given:

```ts
interface OrderItem {
  unitPriceInCents: number;
  quantity: number;
}

interface Order {
  status: "pending" | "completed";
  items: OrderItem[];
}

interface OrderSummary {
  revenueInCents: number;
  itemCount: number;
}
```

Avoid forcing a multi-step summary through `filter()`, `map()`, and `reduce()`:

```ts
function summarizeCompletedOrders(orders: Order[]): OrderSummary {
  return orders
    .filter((order) => order.status === "completed")
    .map((order) => ({
      revenueInCents: order.items.reduce(
        (total, item) => total + item.unitPriceInCents * item.quantity,
        0,
      ),
      itemCount: order.items.reduce(
        (total, item) => total + item.quantity,
        0,
      ),
    }))
    .reduce<OrderSummary>(
      (summary, orderSummary) => ({
        revenueInCents:
          summary.revenueInCents + orderSummary.revenueInCents,
        itemCount: summary.itemCount + orderSummary.itemCount,
      }),
      { revenueInCents: 0, itemCount: 0 },
    );
}
```

Prefer straightforward nested loops when they keep filtering and accumulation visible without intermediate arrays:

```ts
function summarizeCompletedOrders(orders: Order[]): OrderSummary {
  const summary: OrderSummary = {
    revenueInCents: 0,
    itemCount: 0,
  };

  for (const order of orders) {
    if (order.status !== "completed") continue;

    for (const item of order.items) {
      summary.revenueInCents += item.unitPriceInCents * item.quantity;
      summary.itemCount += item.quantity;
    }
  }

  return summary;
}
```
