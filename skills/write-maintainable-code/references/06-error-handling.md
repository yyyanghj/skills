# Handle Errors Deliberately

Treat failure as part of the API contract. Distinguish expected absence from unexpected failure, and handle each error at the closest boundary that can act on it.

## Prefer

- `null`, `undefined`, or a union for expected absence when that matches repository conventions.
- A result type when callers must handle expected failure explicitly.
- Exceptions for failures the current operation cannot recover from.
- Catching at boundaries that own retry, logging, translation, fallback, or user messaging.
- Preserving the original error as a cause when adding context.

## Avoid

- Exceptions used for ordinary branching.
- Catching an error only to return fake success or a vague fallback.
- Converting unrelated failures into the same result.
- Retrying non-idempotent operations without an explicit safety policy.
- Catching before the code can make a meaningful recovery decision.
- Introducing a custom result wrapper when the repository already has a pattern.

Let unexpected failures propagate when the current layer cannot handle them. A top-level handler is useful as a final logging or response boundary, but it should not repair arbitrary failures after the fact.

## Decision Check

- Is this outcome expected absence, recoverable failure, or an invariant violation?
- Can the caller tell what happened and what it may do next?
- Which boundary owns retry, translation, logging, or user communication?
- Are the original error details preserved?

## Example

Suppose `404` means an expected missing user while network and server failures are exceptional.

Avoid collapsing every failure into absence:

```ts
async function getUser(id: string): Promise<User | null> {
  try {
    const response = await fetch(`/api/users/${id}`);
    return parseUser(await response.json());
  } catch {
    return null;
  }
}
```

Encode the contract explicitly:

```ts
async function getUser(id: string): Promise<User | null> {
  const response = await fetch(`/api/users/${id}`);

  if (response.status === 404) {
    return null;
  }

  if (!response.ok) {
    throw new Error(`Failed to load user: HTTP ${response.status}`);
  }

  const payload: unknown = await response.json();
  return parseUser(payload);
}
```
