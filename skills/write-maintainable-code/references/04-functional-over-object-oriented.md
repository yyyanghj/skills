# Prefer Functions over Class Hierarchies

Use functions and plain objects by default. Reach for classes only when there is a real need for stateful lifecycle management, framework integration, or a stable abstraction over external resources.

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
- Default to putting behavior in top-level functions that take the data as the **first parameter**: `processX(x, ...)` instead of `x.process(...)`.
- Group related functions by file or module, not by wrapping them in a class just to "organize" them.
- **Pragmatic exception:** this is a preference, not a prohibition. When a `class` genuinely makes a piece of code more readable and friendly — for example, it manages real internal state, plays nicely with the framework, or encapsulates a resource — use a class. Pick the shape that a reader will understand fastest.

## File Is the Module Boundary

- One file = a related set of types plus the functions that operate on them.
- Share across files via **named exports**, not `export default class XxxService`.
- Achieve privacy by **not exporting** a symbol, not by `private` keywords on class members.

## Express References Directly

- Parent / child, bidirectional, or cyclic references: use pointers/references directly. Do not contort the shape into a tree to look "clean".
- In GC languages, cycles are free — if the domain has one, model it honestly.
- Do not preemptively impose a Rust-style ownership model on designs that do not need it.

## Avoid

- Inheritance hierarchies (especially multi-level or abstract base classes) that exist only to share a few methods.
- Base classes with optional hooks and protected state.
- Constructors that hide important setup work.
- Objects whose only job is to wrap one function call.
- `this`-dependent method chains — unless the object genuinely owns internal state (rare).
- Decorators, dependency-injection frameworks, mixins, and metaprogramming as default tools.

## Classes Are Acceptable When

- A framework or language feature requires them.
- You are managing long-lived state with clear lifecycle rules.
- You are encapsulating a resource such as a socket, file handle, or cache client.
- Polymorphism is solving an active problem with multiple concrete implementations.

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
