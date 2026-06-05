# Apply TypeScript Conventions

Use TypeScript as a tool for clarity, not ceremony. Prefer types that make data flow and failure modes obvious without turning every module into a type puzzle.

## Prefer

- TypeScript-first modules.
- ESM imports and explicit extensions when the runtime requires them.
- Type inference for local variables when the type is obvious.
- Explicit types for exported functions, public APIs, and boundary objects.
- `unknown` plus narrowing over `any`, and prefer simple unions or interfaces over deeply abstracted type systems.

## Avoid

- `any` unless there is a deliberate and temporary escape hatch.
- Complex generic machinery for straightforward data flow.
- Type aliases or interfaces that only restate an obvious local shape once.
- Mixing naming styles within the same area of the codebase.
- Add unnecessary `void` prefixes before function calls.

## Naming Conventions

- Use kebab-case for package directories and files.
- Use camelCase for variables and functions.
- Use PascalCase for components, types, and classes.

## Review Checklist

- Do exported APIs communicate their contract clearly?
- Are unknown values narrowed before use?
- Are the types helping readability, or adding noise?
- Do file and symbol names follow the local conventions?
