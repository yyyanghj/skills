---
name: write-maintainable-code
description: Write solution-focused, simplicity-first code without over-engineering. Use before writing, refactoring, or reviewing code.
---

# Write Maintainable Code

## Core Posture

- **Solution-focused:** Complete the requested task directly. Implement only when implementation is requested; for review tasks, report findings without modifying code unless asked. Be concise and keep changes or explanations limited to decisions that affect the task.
- **Simplicity first:** Choose the simplest design that is clearly correct for the current requirement.
- **Avoid over-engineering:** Add complexity only when a current requirement, concrete boundary, or measured problem demands it.

## Process

1. Inspect the requirement, repository instructions, and the relevant existing code path when one exists. For new code, inspect the target integration point and nearby conventions instead. Finish after reading the instructions that govern every file in scope and either the existing path or, for new code, both the intended integration point and nearby conventions.
2. Load only the references whose conditions match the task. Finish after checking every Reference Map condition and loading each matching reference.
3. Apply Core Posture, Core Rules, Core Avoid, and every loaded reference, then evaluate the result against the Completion Criterion below.

## Priority

Apply guidance in this order:

1. Correctness and current requirements.
2. Repository conventions and existing public contracts.
3. Language and community conventions.
4. Defaults in this skill.

For file names, follow the repository first, then established language or community practice. If neither defines a style, use kebab-case.

## Core Rules

- Optimize for local reasoning: make the path from input to effect easy to trace.
- Keep state, side effects, dynamic wiring, and error paths visible.
- Use domain terms, specific nouns, verb-and-noun operations, and question-form boolean names.
- Name by domain purpose rather than implementation mechanism, and make similar concepts easy to distinguish.
- Use familiar language features by default. Use an advanced feature when it makes the code materially clearer and remains understandable in its local context.

### SOLID

Apply SOLID as a core design discipline across functions, modules, and classes. Use it to keep changes local and contracts clear; do not translate each principle into a new class, interface, or layer.

- **Single Responsibility:** Give each function, module, or class one cohesive responsibility and one primary reason to change.
- **Open/Closed:** Protect stable behavior from repeated edits when real variants exist. Prefer composition or explicit strategy selection; do not create extension points for hypothetical variants.
- **Liskov Substitution:** Require every implementation to honor the full contract, including inputs, outputs, errors, side effects, and lifecycle expectations. Split the abstraction when implementations need exceptions.
- **Interface Segregation:** Expose small interfaces shaped around caller needs. Do not make consumers or implementations depend on unrelated operations.
- **Dependency Inversion:** Keep domain policy independent of volatile infrastructure by passing dependencies explicitly or placing a narrow abstraction at the boundary. Do not add an interface for every concrete dependency.

## Core Avoid

- Over-engineering through speculative abstractions, unused extension points, unnecessary layers or configuration, and unmeasured optimizations.
- Thin wrappers that only rename or forward a call. Keep one only when it isolates a meaningful unstable dependency boundary or enforces an external or public boundary or policy.
- Hidden mutation, implicit registration, swallowed errors, and surprising global state.
- Behavior-changing simplifications presented as refactors.
- Generic placeholder names, unfamiliar abbreviations, overlong names that repeat surrounding module or type context, or comments used only to explain weak identifiers.
- Personal style rules that conflict with the repository or language ecosystem.

## Reference Map

- Read [Simplicity First](references/01-simplicity-first.md) when starting or shaping a function, module, or public API, and when choosing boundaries, layers, interfaces, or reusable abstractions.
- Read [Flat and Linear Control Flow](references/02-flat-and-linear-control-flow.md) when logic is nested, fragmented across helpers or callbacks, or jumps across functions or files.
- Read [Simple and Predictable Functions](references/03-simple-and-predictable-functions.md) when writing, refactoring, or reviewing functions, including business logic, data transforms, state, mutation, effects, hidden dependencies, or helper extraction.
- Read [Data, Objects, Functions, and Classes](references/04-data-objects-functions-and-classes.md) when deciding between plain data, functions, objects, classes, inheritance, and composition.
- Read [Error Handling](references/05-error-handling.md) when calling HTTP services, databases, filesystems, processes, queues, or other external systems, and when adding absence handling, exceptions, retries, fallbacks, or user-facing errors.
- Read [Explicit Dynamic Behavior](references/06-explicit-dynamic-behavior.md) when using registries, string dispatch, metaprogramming, advanced language features, or runtime wiring.
- Read [TypeScript Conventions](references/07-typescript-conventions.md) for TypeScript code, types, exports, functions, and file naming.

## Completion Criterion

The result is complete when it directly satisfies the requested task, matches the surrounding codebase, satisfies Core Posture, Core Rules, and Core Avoid, and satisfies the relevant rules and checks in every loaded reference. Any exception must cite the higher-priority requirement, contract, repository convention, or language/community convention from Priority that justifies it.
