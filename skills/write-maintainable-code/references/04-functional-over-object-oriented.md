# Prefer Functions over Class Hierarchies

Use functions and plain objects by default. Class-based modeling is the exception, not the default. Reach for classes only when there is a real need for stateful lifecycle management, framework integration, or a stable abstraction over external resources.

Think of it as writing C or Go in your target language: **data is data, functions are functions, control flow is control flow**. Treat a language's "advanced features" as occasional tools, not the default unit of organization.

## Use This Rule When

- Choosing between classes, modules, and plain functions.
- Modeling business entities or domain behavior.
- Refactoring object-heavy code that mostly moves data around.

## Prefer

- Plain objects for data.
- Pure or mostly pure functions for behavior.
- Composition over inheritance.
- Small modules with explicit exports.

## Separate Data and Behavior

- Default to describing data shapes with `struct` / `interface` / plain types — fields only, no methods attached.
- Create data with regular or factory functions: `createUser(...)` instead of `new User(...)`.
- Default to putting behavior in top-level functions that take the data as the **first parameter**: `updateUser(user, patch)` instead of `user.update(patch)`.
- Group related functions by file or module, not by wrapping them in a class just to "organize" them.
- **Pragmatic exception:** this is a preference, not a prohibition. When a `class` meets one of the criteria below and makes the code clearer, use it.
- Treat a justified class like a Go or Rust struct with methods: a container for state and invariants, not an inheritance-based domain model.

## File Is the Module Boundary

- One file = a related set of types plus the functions that operate on them.
- Share across files via **named exports**, not `export default class XxxService`.
- Achieve privacy by **not exporting** a symbol, not by `private` keywords on class members.

## Express References Directly

- Parent / child, bidirectional, or cyclic references: use pointers/references directly. Do not contort the shape into a tree to look "clean".
- In GC languages, cycles are free — if the domain has one, model it honestly.
- Do not preemptively impose a Rust-style ownership model on designs that do not need it.

## Avoid

- Inheritance hierarchies, abstract base classes, parent-class hooks, template methods, and `is-a` taxonomies; prefer composition and explicit dependency passing.
- Constructors that hide important setup work.
- Objects whose only job is to wrap one function call.
- `this`-dependent method chains — unless the object genuinely owns internal state (rare).
- Decorators, dependency-injection frameworks, mixins, and metaprogramming as default tools.

## Classes Are Acceptable When

- A framework or language feature requires them.
- The object has meaningful identity or lifecycle rules.
- The object owns internal mutable state or a cache shared across operations.
- The object manages a resource such as a connection, file handle, transaction, or subscription, including its `dispose` or cleanup behavior.
- Private implementation details or invariants must remain consistent across operations.

## Review Checklist

- Is this stateful enough to justify an instance?
- Would plain functions plus explicit data be easier to test?
- Are we using inheritance where composition would be simpler?
- Can the behavior be described without an object identity?

## Example

**Bad:**

```ts
class Animal {
  constructor(public name: string) {}

  eat() {
    console.log(`${this.name} is eating.`);
  }
}

class Dog extends Animal {
  bark() {
    console.log(`${this.name} is barking.`);
  }
}

class Cat extends Animal {
  meow() {
    console.log(`${this.name} is meowing.`);
  }
}

const zoo = new Zoo();
zoo.addAnimal(new Dog("Buddy"));
zoo.feedAnimals();
```

**Good:**

```ts
interface Animal {
  name: string;
}

function eat(animal: Animal) {
  console.log(`${animal.name} is eating.`);
}

function bark(dog: Animal) {
  console.log(`${dog.name} is barking.`);
}

const buddy: Animal = { name: "Buddy" };
eat(buddy);
```
