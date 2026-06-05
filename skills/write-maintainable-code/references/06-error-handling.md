# Handle Errors Deliberately

Treat error handling as part of the contract, not an afterthought. Make failure modes explicit, keep recovery near the boundary that can handle it, and do not use exceptions as normal control flow.

## Use This Rule When

- Calling risky external systems.
- Deciding whether to throw, return a result object, or return `null`.
- Adding retries, fallbacks, or user-facing error messages.

## Prefer

- Normal return values for expected absence or branch conditions.
- A project-standard result wrapper such as `tryCatch()` when one exists.
- Letting unexpected failures bubble to the boundary that can log or translate them.
- Adding context when rethrowing so the failure is diagnosable.

## Avoid

- `try/catch` blocks used only to steer normal branching.
- Swallowing an error and returning a fake success.
- Converting every failure into the same vague message.
- Catching too early, before the code can handle the problem meaningfully.

## Decision Guide

- Return `null` or a union type when absence is expected.
- Return a result object when callers should handle failure explicitly.
- Throw when the caller cannot reasonably continue without intervention.
- Catch near the boundary that owns logging, retries, translation, or user messaging.

## Review Checklist

- Is this an exceptional failure or a normal branch?
- Can the caller tell what failed and what to do next?
- Are we preserving the original error details?
- Is the catch block actually handling the problem, or just hiding it?

## Example

**Bad:**

```ts
try {
  const value = await api.fetchData();
  return value;
} catch (error) {
  return handleError(error);
}
```

**Good:**

```ts
const result = await tryCatch(() => api.fetchData());

if (!result.ok) {
  return handleError(result.error);
}

return result.value;
```

If the project does not provide a `tryCatch()` helper, use the existing result pattern in the codebase or let the exception propagate to a boundary that can handle it correctly.
