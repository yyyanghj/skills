# Keep Control Flow Flat and Linear

Keep execution flat, linear, and local. A reader should be able to trace the path from entry point to effect from top to bottom without repeatedly jumping across thin functions, callbacks, or files.

## Rules

### Prefer

- Guard clauses for invalid state and edge cases.
- One visible orchestrator that calls leaf operations in execution order.
- Direct local variables, loops, and branches.
- State changes close to the conditions that trigger them.
- A few meaningful stages instead of many one-call steps.

### Avoid

- Deeply nested `if`, `else`, or callback chains.
- Forcing readers to bounce across functions or files to reconstruct one synchronous flow.
- Callback parameters whose only purpose is to continue execution elsewhere.
- Dispatcher chains that forward to another dispatcher.
- Hiding important branches behind vague helper names.

### Review Check

- Can a reader follow the main path from top to bottom?
- Are important branches and state changes visible near their effects?
- Does each helper own meaningful logic or policy, or isolate a meaningful unstable dependency boundary?
- Would inlining a callback or forwarding layer make execution easier to trace?
- Is the happy path visually dominant?

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

Prefer guard clauses with the same failure behavior:

```ts
function saveUser(user: User | null): void {
  if (!user) throw new Error("User is required");
  if (!user.isActive) throw new Error("User is inactive");
  if (!user.canEdit) throw new Error("User cannot edit");

  persistUser(user);
}
```
