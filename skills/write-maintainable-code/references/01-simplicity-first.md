# Simplicity First

Use these rules to choose function, module, API, and abstraction shape.

Prefer boring code over impressive code when both solve the problem clearly.

## Avoid Over-Engineering

Do not add abstractions, layers, extension points, configuration, or optimizations for hypothetical future needs. Add complexity only when a current requirement, real boundary, or measured problem justifies it.

Examples:

- **Single implementation:** Call the concrete function directly instead of adding an interface, factory, or registry for implementations that do not exist.
- **Real dependency boundary:** Use one focused adapter to isolate an unstable SDK or external API; do not surround it with service, repository, mapper, and DTO layers unless each layer owns distinct behavior.
- **Unproven reuse:** Keep a small amount of duplication while similar code still changes for different reasons. Extract shared logic after repeated changes reveal a stable concept.
- **Performance:** Measure the bottleneck before adding caches, batching, workers, or concurrency controls.

An abstraction with one caller is still justified when it enforces a public contract, isolates an unstable dependency, or owns cross-cutting policy.

## Prefer

- Concrete types and direct return shapes.
- Small public APIs and visible data flow.
- Existing language features before custom frameworks.
- Temporary duplication while a shared shape is still changing.
- Incremental extraction after repeated change pressure appears.

## Avoid

- Abstract types, factories, or extension points with one speculative use.
- Service, repository, mapper, or DTO layers with no real boundary or transformation.
- Configuration options that no current caller needs.
- Indirection added only to look scalable.
- Optimization without a measured bottleneck.

## Decision Check

- What current pain does the abstraction remove?
- Does it remove repeated changes rather than merely repeated lines?
- Is the happy path easier to understand with the layer present?
- Would deleting the layer lose a real boundary, policy, or contract?

## Example

Given:

```ts
interface UserInput {
  username: string;
  password: string;
}
```

Avoid a rule engine when two fixed checks are the whole current requirement:

```ts
interface ValidationRule {
  validate(input: UserInput): string | null;
}

class RequiredUsernameRule implements ValidationRule {
  validate(input: UserInput): string | null {
    return input.username.trim() ? null : "Username is required";
  }
}

class MinimumPasswordLengthRule implements ValidationRule {
  validate(input: UserInput): string | null {
    return input.password.length >= 8
      ? null
      : "Password must be at least 8 characters";
  }
}

function validateUser(input: UserInput, rules: ValidationRule[]): string[] {
  return rules.flatMap((rule) => {
    const error = rule.validate(input);
    return error ? [error] : [];
  });
}
```

Prefer the direct implementation until callers need configurable rules:

```ts
function validateUser(input: UserInput): string[] {
  const errors: string[] = [];

  if (!input.username.trim()) {
    errors.push("Username is required");
  }

  if (input.password.length < 8) {
    errors.push("Password must be at least 8 characters");
  }

  return errors;
}
```
