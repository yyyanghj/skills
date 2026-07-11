# Explicit Dynamic Behavior

Prefer syntax and control flow that maintainers can understand locally. Keep behavior selection, wiring, and runtime effects visible.

## Portability

Treat portability as a tiebreaker, not a ban. When alternatives are equally clear, choose familiar constructs with explicit semantics that transfer across languages. Use a language-specific or less common feature when it materially improves clarity, removes more complexity than it adds, fits the repository and toolchain, and can remain narrow and conventional.

## Prefer

- Direct calls and visible dependency wiring.
- Declarative configuration with explicit execution.
- Plain data, functions, enums or tagged unions, branches, and loops when they express the behavior clearly.
- Validation where untyped or external data enters the system.
- Explicit rejection of unknown cases instead of undefined behavior.
- Exhaustive branches for closed sets of variants.
- Lookup tables when the mapping is stable, validated, and clearer than branching.

## Avoid

- Registration through import side effects.
- String-based dispatch without input validation.
- Decorators or metaprogramming that hide ownership or execution order.
- Runtime magic such as prototype mutation, metaclasses, or dynamic method synthesis when a direct construct would work.
- Runtime-heavy pipelines when a direct branch or loop is clearer.
- Type-level machinery that is harder to understand than the data it describes.

## Model Closed Variants Explicitly

Use a tagged union or the language's equivalent when a value can be one of a fixed set of shapes. Keep dispatch in one visible place when doing so makes the behavior easier to inspect.

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

## Review Check

- Can a reader see how behavior is selected and invoked?
- Is dynamic input validated before dispatch?
- Do unknown cases fail explicitly and diagnostically?
- Does the advanced feature reduce total cognitive load here?
- When alternatives are equally clear, does the design avoid unnecessary runtime-specific semantics?
- Would a maintainer need unusual language knowledge to change this safely?
