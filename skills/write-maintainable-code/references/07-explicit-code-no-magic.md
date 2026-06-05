# Keep Code Explicit

Prefer obvious control flow and validated inputs over clever indirection. A reader should not need to guess how behavior is discovered, wired, or transformed at runtime.

## Use This Rule When

- Mapping strings to behaviors.
- Introducing registries, plugin systems, or dynamic lookup tables.
- Considering metaprogramming, decorators, or side-effect imports.

## Prefer

- Explicit branches for small and medium decision sets.
- Exhaustive `switch` statements for closed unions.
- Validation close to dynamic input boundaries.
- Direct function calls and visible wiring.

## Avoid

- Hidden registration through module side effects.
- Stringly typed dispatch without validation.
- Meta-programming that obscures ownership or control flow.
- Clever shorthand that saves lines but hides behavior.
- Runtime-heavy control-flow abstractions (long Promise chains, Observables, Generators, reactive pipelines) when a plain `if` / `for` / `switch` would work. You should be able to read the code on paper, without simulating a runtime in your head.
- Global error handlers used to "clean up" fallout. Errors belong in return values or explicit throws at the point that can handle them.
- Deep generic machinery, exotic conditional types, and other type-level puzzles when a plain type would do. Types should aid reading, not require decoding.

## Model Variants with Tagged Unions + Switch

- Give each variant a discriminator field (`kind`, `type`, or `tag`) whose value is a literal / enum.
- Express polymorphism as a **single central `switch` on the tag**, not by overriding methods on subclasses.
- Benefits: exhaustiveness checking is easy, the call stack stays flat, and the pattern ports cleanly across languages.

**Bad — dispatch scattered across a class hierarchy:**

```ts
abstract class Shape {
  abstract area(): number;
}
class Circle extends Shape {
  constructor(public r: number) { super(); }
  area() { return Math.PI * this.r ** 2; }
}
class Square extends Shape {
  constructor(public side: number) { super(); }
  area() { return this.side ** 2; }
}
```

**Good — tagged union, one switch:**

```ts
type Shape =
  | { kind: "circle"; r: number }
  | { kind: "square"; side: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle": return Math.PI * shape.r ** 2;
    case "square": return shape.side ** 2;
  }
}
```

## Write Portable Code

- Avoid language-specific magic: JS prototype tricks, Python metaclasses, Ruby `method_missing`, decorators that rewrite semantics, etc.
- Assume the code might need to be translated into another systems language tomorrow. Prefer constructs every language has: struct, function, enum, switch, pointer / reference, loop.
- If a feature only works because of one language's runtime behavior, that is a smell — prefer the plain equivalent.

## Practical Guidance

- Use lookup maps only when the mapping is stable, validated, and truly clearer.
- Fail loudly on unknown cases instead of returning undefined behavior.
- Keep configuration declarative, but keep execution explicit.
- Prefer readable branching over compact tricks.

## Review Checklist

- Can a reader see every valid branch in one place?
- What happens for invalid input, and is that path explicit?
- Are we depending on import order or hidden registration?
- Would a straightforward `switch` or function call be clearer?

## Example

**Bad:**

```ts
const formControls = {
  text: () => new TextInput(),
  number: () => new NumberInput(),
};

function renderFormControl(type: string) {
  return formControls[type]();
}
```

**Good:**

```ts
function renderFormControl(type: FormControlType) {
  switch (type) {
    case "text":
      return new TextInput();
    case "number":
      return new NumberInput();
    default:
      throw new Error(`Unknown type: ${type}`);
  }
}
```
