---
name: write-maintainable-code
description: Write solution-focused, simplicity-first code. Use before writing, refactoring, or reviewing code.
---

# Write Maintainable Code

## Core Posture

- **Solution-focused:** Complete the requested task directly. Implement only when implementation is requested; for review tasks, report findings without modifying code unless asked. Be concise and keep changes or explanations limited to decisions that affect the task.
- **Simplicity first:** Choose the simplest design that is clearly correct for the current requirement. Add complexity only when a present constraint makes the simpler option inadequate.

## Process

1. Inspect the requirement, existing implementation, nearby code, and repository instructions. Finish after reading the code path being changed or reviewed and the instructions that govern every file in scope.
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
- Use familiar language features by default. Use an advanced feature when it makes the code materially clearer and remains understandable in its local context.

## Core Avoid

- Speculative abstractions, unused extension points, and unmeasured optimizations.
- Thin wrappers that only rename or forward a call. Keep one only when it enforces a meaningful external or public boundary or policy.
- Hidden mutation, implicit registration, swallowed errors, and surprising global state.
- Behavior-changing simplifications presented as refactors.
- Generic placeholder names, unfamiliar abbreviations, or comments used only to explain weak identifiers.
- Personal style rules that conflict with the repository or language ecosystem.

## Reference Map

- Read [Put Simplicity First](references/01-simplicity-first.md) when starting or shaping a function, module, or public API, and when choosing boundaries, layers, interfaces, or reusable abstractions.
- Read [Keep Control Flow and Functions Flat and Linear](references/02-control-flow-and-functions.md) when logic is nested, fragmented across helpers or callbacks, jumps across files, is stateful, or mixes calculation with effects.
- Read [Choose Data, Functions, and Classes Deliberately](references/03-data-functions-and-classes.md) when deciding between plain data, functions, objects, classes, inheritance, and composition.
- Read [Define Error Contracts](references/04-error-contracts.md) when calling HTTP services, databases, filesystems, processes, queues, or other external systems, and when adding absence handling, exceptions, retries, fallbacks, or user-facing errors.
- Read [Keep Dynamic Behavior Explicit](references/05-explicit-dynamic-behavior.md) when using registries, string dispatch, metaprogramming, advanced language features, or runtime wiring.
- Read [Apply TypeScript Conventions](references/06-typescript.md) for TypeScript code, types, exports, functions, and file naming.

## Completion Criterion

The result is complete when it directly solves the requested problem with the simplest clearly correct design, matches the surrounding codebase, satisfies Core Rules and Core Avoid, and satisfies the relevant rules and checks in every loaded reference. Any exception must cite the higher-priority requirement, contract, repository convention, or language/community convention from Priority that justifies it.
