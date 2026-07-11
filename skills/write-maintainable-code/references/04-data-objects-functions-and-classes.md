# Data, Objects, Functions, and Classes

Prefer plain data and functions by default. Apply the criteria in `Use a Class When` before introducing a class.

## Prefer

- Plain objects for data.
- Functions for stateless behavior and transformations.
- Regular or factory functions for constructing plain data.
- Module-level behavior functions that take the primary data as the first parameter, such as `updateUser(user, patch)`.
- Related data types and functions grouped in a module instead of a class used only as a namespace.
- Modules as the default organization and privacy boundary; use class privacy when a justified class owns invariants.
- In JavaScript and TypeScript, use named exports for cross-file APIs; avoid default-exported service classes used only as module containers.
- Composition and explicit dependency passing over inheritance.
- Modules that expose a small set of intentional operations.

## Avoid

- Inheritance hierarchies used only to organize related behavior.
- Abstract base classes with one implementation.
- `this`-dependent method chains when the object does not own meaningful state.
- Constructors that hide I/O or other important setup work.
- Decorators, dependency-injection frameworks, or mixins used as default organization tools.
- Cyclic object graphs introduced only for convenience.

Use direct references when the domain is genuinely cyclic, but account for object lifetime, serialization, debugging, and retained memory. Garbage collection can reclaim unreachable cycles; it does not make cycles free.

## Use a Class When

- A framework or language feature requires one.
- The object has meaningful identity or lifecycle rules.
- Operations share internal mutable state or a cache.
- The object manages cleanup for a connection, file, transaction, or subscription.
- Encapsulation keeps invariants consistent across operations.

## Review Check

- Does this instance own state or lifecycle, or is it only a namespace?
- Would a plain value plus functions be easier to understand and test?
- Does inheritance add substitutability, or only indirect dispatch?
- Are relationships modeled honestly without ignoring lifetime costs?

## Example

Avoid a class that only groups stateless behavior:

```ts
class UserFormatter {
  formatDisplayName(user: User): string {
    return `${user.firstName} ${user.lastName}`;
  }
}

const formatter = new UserFormatter();
const displayName = formatter.formatDisplayName(user);
```

Prefer a function with the same behavior:

```ts
function formatDisplayName(user: User): string {
  return `${user.firstName} ${user.lastName}`;
}

const displayName = formatDisplayName(user);
```
