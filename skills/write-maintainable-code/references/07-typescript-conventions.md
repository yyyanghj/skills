# TypeScript Conventions

Use TypeScript to make data shapes, public contracts, and failure modes explicit without turning straightforward code into type-level machinery. Prefer familiar types that keep runtime data flow easy to follow.

Use `interface` by default for object-shaped data and public object contracts. Use `type` when the definition requires type operations or forms that an interface does not express directly, such as unions, intersections, tuples, mapped types, or conditional types.

Apply the Priority in `SKILL.md` when repository, language, or framework conventions conflict with these defaults.

## Prefer

- `interface` for object-shaped data and public object contracts.
- `type` for unions, intersections, tuples, mapped types, conditional types, and other type-level composition.
- Type inference for obvious local values.
- Explicit types for exported APIs and boundary objects.
- `unknown` plus narrowing for untrusted values.
- Simple unions and interfaces before complex generics.
- `function` declarations for file- and module-level functions unless a higher-priority repository or framework convention requires otherwise.
- Arrow functions for callbacks and expression-oriented APIs.
- Named exports for cross-file APIs.
- ESM imports and explicit extensions when required by the runtime.

## Avoid

- `type` aliases for plain object shapes when an `interface` expresses the contract directly.
- `any` unless it is a deliberate, temporary escape hatch with a clear removal path.
- Type assertions used instead of validating external data.
- Deep conditional types or generic machinery for straightforward data flow.
- One-use aliases or interfaces that only repeat an obvious local shape.
- Arrow-function expressions for module-level functions without a repository or framework reason.
- Default-exported service classes used only as module containers.
- Unnecessary `void` prefixes before function calls.
- Naming changes that create churn without improving clarity.

## Design and Review Check

Use these questions while designing TypeScript APIs and reviewing the implementation:

- Is an object-shaped contract expressed as an `interface` by default?
- Does each `type` alias represent a union, composition, or real type operation?
- Are exported APIs and untrusted boundaries typed explicitly?
- Is `unknown` narrowed before external data is used?
- Could a complex generic be replaced with a direct data shape?
- Do module-level functions follow the repository's declaration style and this skill's default?
- Is every use of `any` temporary and paired with a removal path?

## Examples

### Example: Interface for Data Shapes

Avoid using `type` for ordinary object contracts without a reason:

```ts
type User = {
  id: string;
  name: string;
};

type UserResponse = {
  user: User;
  requestId: string;
};
```

Prefer `interface` for object shapes and reserve `type` for type composition:

```ts
interface User {
  id: string;
  name: string;
}

interface UserResponse {
  user: User;
  requestId: string;
}

type UserState = "active" | "disabled";
```

### Example: Validate Untrusted Data

Avoid allowing untrusted data to bypass the type system:

```ts
function readUserName(payload: any): string {
  return payload.user.name;
}
```

Prefer narrowing `unknown` at the boundary:

```ts
function readUserName(payload: unknown): string {
  if (!payload?.user || typeof payload?.user !== 'object') {
    throw new Error("Expected payload.user to be an object");
  }

  if (typeof payload.user.name !== "string") {
    throw new Error("Expected payload.user.name to be a string");
  }

  return payload.user.name;
}
```

If validation is repeated or the schema is complex, use the repository's existing schema-validation library instead of duplicating manual checks.
