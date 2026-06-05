---
name: write-maintainable-code
description: Core principles for writing simple, readable, and maintainable code. You MUST use this before generating, refactoring, or reviewing code to keep implementations simple, readable, explicit, and easy to maintain.
---

> Be concise, direct, and solution-focused.

# Write Maintainable Code

Use this skill as a lightweight entry point. Read only the reference files that match the task instead of loading the entire rule set every time.

## Default posture

- Prefer the simplest code that solves the current requirement.
- Keep control flow flat, explicit, and easy to trace.
- Add abstractions only after repeated pressure, not in anticipation.
- Do not add helper functions that only rename, forward, or wrap a single call without removing real complexity.
- Favor plain data and functions over class hierarchies.
- Keep names, types, and error paths unsurprising.

## Reference map

- Read [Prefer Simplicity First](references/01-simplicity-first.md) when choosing the overall shape of a solution.
- Read [Keep Logic Flat and Linear](references/02-flat-and-linear-logic.md) when flow control is becoming nested or indirect.
- Read [Avoid Over-Engineering](references/03-no-over-engineering.md) when you are tempted to add layers, interfaces, or future-proofing.
- Read [Prefer Functions over Class Hierarchies](references/04-functional-over-object-oriented.md) when deciding between functions, objects, classes, and composition.
- Read [Keep Functions Simple and Predictable](references/05-simple-and-predictable-functions.md) when writing business logic or data transforms.
- Read [Handle Errors Deliberately](references/06-error-handling.md) when adding failure handling, retries, recovery, or wrappers.
- Read [Keep Code Explicit](references/07-explicit-code-no-magic.md) when dynamic behavior, implicit wiring, or meta-programming is involved.
- Read [Name Things Clearly](references/08-naming-rules.md) when choosing identifiers or reviewing clarity.
- Read [Apply TypeScript Conventions](references/09-typescript-guidelines.md) for TypeScript-specific practices.

## Suggested read order

- If the task is small or ambiguous, start with `01`, `02`, and `08`.
- If the task is mainly about API or architecture shape, start with `01`, `03`, and `04`.
- If the task is mainly about implementation detail, read `05`, `06`, `07`, and `09` as needed.
