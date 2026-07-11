# Data, Objects, Functions, and Classes

Separate data and behavior by default. Keep data in plain values or objects and implement stateless behavior with standalone functions. Do not upgrade plain data into a class merely because operations validate, normalize, resolve, or transform it.

Use a class only when an instance must own state, identity, lifecycle, resources, or invariants across operations. Treat a justified class like a Go or Rust struct with methods: a container for state and invariants, not an inheritance-based domain model.

## Prefer

- Plain objects for data and standalone functions for behavior.
- Regular or factory functions for constructing values.
- Behavior functions that take the primary value first, such as `updateUser(user, patch)`.
- Related data types and functions grouped in a module with a small public API.
- A class when an instance owns meaningful identity or lifecycle rules.
- A class for internal mutable state, caches, subscriptions, connections, transactions, or resources that require cleanup.
- Class encapsulation when it must keep private state and invariants consistent across operations.
- Composition and explicit dependency passing over inheritance.

## Avoid

- Classes used only as namespaces or containers for plain data.
- `new User(...)` and instance methods when plain values and functions express the same model directly.
- Inheritance hierarchies, abstract base classes, parent hooks, template methods, or `is-a` classification models.
- Polymorphism introduced before real substitutable implementations exist.
- Constructors that hide I/O or other important setup work.
- Decorators, dependency-injection frameworks, or mixins used as default organization tools.
- Cyclic object graphs introduced only for convenience; when the domain is genuinely cyclic, account for lifetime, serialization, debugging, and retained memory.

## Design and Review Check

Use these questions while designing the model and reviewing the implementation:

- Are data shapes visible and separate from stateless behavior?
- Would a plain value plus standalone functions be easier to understand and test?
- If a class is proposed, what state, identity, lifecycle, resource, or invariant does the instance own?
- Does the class look like a state container with methods, or the root of an inheritance model?
- Are dependencies and I/O explicit rather than hidden in construction or decorators?
- Could composition replace inheritance without losing a real contract?

## Examples

### Example: Options Stay as Plain Data

Given external options with this shape:

```ts
interface BuildOptions {
  rootDir: string;
  minify?: boolean;
}

interface ResolvedConfig {
  rootDir: string;
  minify: boolean;
}

declare const input: BuildOptions;
```

Avoid upgrading external options into a class only to attach stateless operations:

```ts
class BuildOptions {
  constructor(
    readonly rootDir: string,
    readonly minify?: boolean,
  ) {}

  validate(): string[] {
    return this.rootDir.trim() ? [] : ["rootDir is required"];
  }

  resolve(): ResolvedConfig {
    return {
      rootDir: this.rootDir,
      minify: this.minify ?? false,
    };
  }
}

const options = new BuildOptions(input.rootDir, input.minify);
const errors = options.validate();
const config = options.resolve();
```

Prefer keeping the options as data and passing them to explicit behavior functions:

```ts
function validateOptions(options: BuildOptions): string[] {
  return options.rootDir.trim() ? [] : ["rootDir is required"];
}

function resolveConfig(options: BuildOptions): ResolvedConfig {
  return {
    rootDir: options.rootDir,
    minify: options.minify ?? false,
  };
}

const errors = validateOptions(input);
const config = resolveConfig(input);
```

### Example: Class with Lifecycle State

Avoid separate functions plus shared mutable state for a resource lifecycle:

```ts
let socket: WebSocket | null = null;

function connect(url: string): void {
  socket = new WebSocket(url);
}

function close(): void {
  socket?.close();
  socket = null;
}
```

Prefer a class when each instance owns the resource and its invariant:

```ts
class WebSocketSession {
  private socket: WebSocket | null = null;

  constructor(private readonly url: string) {}

  connect(): void {
    if (this.socket) return;
    this.socket = new WebSocket(this.url);
  }

  close(): void {
    this.socket?.close();
    this.socket = null;
  }
}
```
