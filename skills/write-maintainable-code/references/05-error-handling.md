# Error Handling

Treat failure as part of the API contract. Distinguish expected absence, recoverable failure, and unexpected failure so callers can tell what happened and what they may do next.

Handle an error at the closest boundary that can make a meaningful decision about retry, translation, fallback, logging, or user communication. Otherwise preserve the original failure and let it propagate.

## Prefer

- `null`, `undefined`, or a union for expected absence when that matches repository conventions.
- A result type when callers must handle an expected failure explicitly.
- Exceptions for failures the current operation cannot recover from.
- Catching at the boundary that owns retry, translation, fallback, logging, or user messaging.
- Preserving the original error as `cause` when adding context.
- Explicit retry policies that account for idempotency, limits, and backoff.

## Avoid

- Exceptions used for ordinary branching.
- Catching an error only to return fake success, absence, or a vague fallback.
- Converting unrelated failures into the same result.
- Retrying non-idempotent operations without an explicit safety policy.
- Catching before the code can make a meaningful recovery decision.
- Swallowing the original error while adding context.
- Introducing a custom result wrapper when the repository already has a pattern.

## Design and Review Check

Use these questions while designing the error contract and reviewing the implementation:

- Is the outcome expected absence, recoverable failure, or an invariant violation?
- Can the caller distinguish the relevant failure cases?
- Which boundary owns retry, translation, fallback, logging, or user communication?
- Can the current layer actually recover, or should the error propagate?
- Are the original error details preserved?
- Is every retry safe for the operation being repeated?

## Examples

### Example: Absence Is Not Every Failure

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

Prefer encoding the contract explicitly:

```ts
async function getUser(id: string): Promise<User | null> {
  const response = await fetch(`/api/users/${id}`);

  if (response.status === 404) return null;
  if (!response.ok) {
    throw new Error(`Failed to load user: HTTP ${response.status}`);
  }

  const payload: unknown = await response.json();
  return parseUser(payload);
}
```

### Example: Preserve the Original Cause

Avoid replacing a diagnostic failure with a context-free error:

```ts
async function loadConfig(path: string): Promise<Config> {
  try {
    return parseConfig(await readFile(path, "utf8"));
  } catch {
    throw new Error("Failed to load config");
  }
}
```

Prefer adding useful context while preserving the cause:

```ts
async function loadConfig(path: string): Promise<Config> {
  try {
    return parseConfig(await readFile(path, "utf8"));
  } catch (error) {
    throw new Error(`Failed to load config: ${path}`, { cause: error });
  }
}
```
