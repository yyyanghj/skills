# Keep Logic Flat and Linear

Keep the abstraction layer thin. A reader should be able to trace the path from entry point to effect without jumping through a stack of tiny wrappers.

## Use This Rule When

- Control flow starts nesting several levels deep.
- A feature is spread across many helpers that each do very little.
- Failure paths are harder to see than the happy path.

## Prefer

- Guard clauses that reject invalid input early.
- Short loops and direct local variables.
- One level of indirection when it clearly improves reuse.
- State changes that happen close to the conditions that trigger them.

## Avoid

- Arrow code with nested `if`, `else`, and callbacks.
- Dispatcher chains that forward to another dispatcher.
- Small wrappers that only rename or pass through data.
- Meaningless helper functions that hide a single obvious call behind another name.
- Hiding important branching behind helpers with vague names.

## Tactics

- Return early for invalid state and edge cases.
- Collapse one-use helper chains back into the caller.
- Split long flows into a few named stages, not many microscopic steps.
- Keep the happy path visually dominant.

## Review Checklist

- Can you follow the main path from top to bottom?
- Are edge cases handled before the core work starts?
- Is each helper doing real work, or only forwarding?
- Would inlining a layer make the code easier to reason about?

## Heuristic: Inline Thin Helpers

If a helper only renames a call, forwards arguments unchanged, or wraps one obvious expression, inline it at the call site. Extract a helper only when it removes real branching, repeated change pressure, or domain complexity.

**Bad:**

```ts
export function resolveReleaseChangesets(output: string) {
  return toChangesetFiles(output);
}

export function getPendingChangesets({ sha }: { sha: string }) {
  return resolveReleaseChangesets(
    run(`git ls-tree -r --name-only ${sha} .changeset`),
  );
}

const addedChangesets = getPendingChangesets({ sha });
```

**Good:**

```ts
const addedChangesets = toChangesetFiles(
  run(`git ls-tree -r --name-only ${sha} .changeset`),
);
```

## Example: Avoid Excessive Indirection

**Bad:**

```ts
function dispatchEvent(nodeKey: string, eventName: string) {
  const node = getNode(nodeKey);
  dispatchNodeEvent(node, eventName);
}

function dispatchNodeEvent(node: Node, eventName: string) {
  const actions = getNodeActions(node, eventName);
  dispatchActions(actions);
}

function dispatchActions(actions: Action[]) {
  for (const action of actions) {
    handleAction(action);
  }
}

function handleAction(action: Action) {
  const ctx = createContext(action);
  action.handle(ctx);
}
```

**Good:**

```ts
function dispatchEvent(nodeKey: string, eventName: string) {
  const node = getNode(nodeKey);
  const actions = node.events[eventName] || [];

  for (const action of actions) {
    const ctx = createContext(action);
    action.handle(ctx);
  }
}
```

## Example: Prefer Guard Clauses

**Bad:**

```ts
function processUser(user: User | null) {
  if (user) {
    if (user.isActive) {
      if (user.hasPermission) {
        saveData(user);
      } else {
        throw new Error("No permission");
      }
    } else {
      throw new Error("User inactive");
    }
  } else {
    throw new Error("No user");
  }
}
```

**Good:**

```ts
function processUser(user: User | null) {
  if (!user) throw new Error("No user");
  if (!user.isActive) throw new Error("User inactive");
  if (!user.hasPermission) throw new Error("No permission");

  saveData(user);
}
```
