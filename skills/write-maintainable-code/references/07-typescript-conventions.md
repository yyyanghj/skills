# TypeScript Conventions

Use TypeScript to clarify data flow, public contracts, and failure modes. Apply the Priority in `SKILL.md` when guidance conflicts.

## Prefer

- Type inference for obvious local values.
- Explicit types for exported APIs and boundary objects.
- `unknown` plus narrowing for untrusted values.
- Simple unions and interfaces before complex generics.
- Define file- and module-level functions with `function` declarations, not arrow-function expressions, unless a higher-priority repository or framework convention requires otherwise.
- Arrow functions for callbacks and when the repository or framework expects expressions.
- ESM imports and explicit extensions when required by the runtime.

## Avoid

- `any` unless it is a deliberate, temporary escape hatch with a clear removal path. Use `unknown` plus validation for permanent untyped boundaries.
- Type assertions used instead of validating external data.
- Deep conditional types or generic machinery for straightforward data flow.
- One-use aliases or interfaces that only repeat an obvious local shape.
- Unnecessary `void` prefixes before function calls.
- Naming changes that create churn without improving clarity.

For identifiers and file names, apply the corresponding rules and priority in `SKILL.md`.

## Example

Avoid allowing untrusted data to bypass the type system:

```ts
function readUserName(payload: any): string {
  return payload.user.name;
}
```

Validate the boundary before using the value:

```ts
function readUserName(payload: unknown): string {
  if (
    typeof payload !== "object" ||
    payload === null ||
    !("user" in payload) ||
    typeof payload.user !== "object" ||
    payload.user === null ||
    !("name" in payload.user) ||
    typeof payload.user.name !== "string"
  ) {
    throw new Error("Invalid user payload");
  }

  return payload.user.name;
}
```

If validation is repeated or the schema is complex, use the repository's existing schema-validation library instead of duplicating manual checks.
