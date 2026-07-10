---
name: write-maintainable-code
description: Write solution-focused, simplicity-first code. Use before writing, refactoring, or reviewing code.
---

# Write Maintainable Code

## Core Posture

- **Solution-focused:** Complete the requested task directly. Implement only when implementation is requested; for review tasks, report findings without modifying code unless asked. Be concise and keep changes or explanations limited to decisions that affect the task.
- **Simplicity first:** Choose the simplest design that is clearly correct for the current requirement. Add complexity only when a present constraint makes the simpler option inadequate.

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

## Core Avoid

- Speculative abstractions, unused extension points, and unmeasured optimizations.
- Thin wrappers that only rename or forward a call. Keep one only when it isolates a meaningful unstable dependency boundary or enforces an external or public boundary or policy.
- Hidden mutation, implicit registration, swallowed errors, and surprising global state.
- Behavior-changing simplifications presented as refactors.
- Generic placeholder names, unfamiliar abbreviations, overlong names that repeat surrounding module or type context, or comments used only to explain weak identifiers.
- Personal style rules that conflict with the repository or language ecosystem.

## Reference Map

- Read [Put Simplicity First](references/01-simplicity-first.md) when starting or shaping a function, module, or public API, and when choosing boundaries, layers, interfaces, or reusable abstractions.
- Read [Keep Control Flow Flat and Linear](references/02-flat-and-linear-logic.md) when logic is nested, fragmented across helpers or callbacks, or jumps across functions or files.
- Read [Avoid Excessive Indirection](references/03-excessive-indirection-example.md) when tracing a flow requires following callback, dispatcher, or helper chains across functions or files.
- Read [Keep Functions Simple and Predictable](references/04-simple-and-predictable-functions.md) when writing, refactoring, or reviewing functions, including business logic, data transforms, state, mutation, effects, hidden dependencies, or helper extraction.
- Read [Choose Data, Functions, and Classes Deliberately](references/05-data-functions-and-classes.md) when deciding between plain data, functions, objects, classes, inheritance, and composition.
- Read [Handle Errors Deliberately](references/06-error-handling.md) when calling HTTP services, databases, filesystems, processes, queues, or other external systems, and when adding absence handling, exceptions, retries, fallbacks, or user-facing errors.
- Read [Keep Dynamic Behavior Explicit](references/07-explicit-code-no-magic.md) when using registries, string dispatch, metaprogramming, advanced language features, or runtime wiring.
- Read [Apply TypeScript Conventions](references/08-typescript-guidelines.md) for TypeScript code, types, exports, functions, and file naming.

## Completion Criterion

The result is complete when it directly satisfies the requested task, matches the surrounding codebase, satisfies Core Posture, Core Rules, and Core Avoid, and satisfies the relevant rules and checks in every loaded reference. Any exception must cite the higher-priority requirement, contract, repository convention, or language/community convention from Priority that justifies it.
