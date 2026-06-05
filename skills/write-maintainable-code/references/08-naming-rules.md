# Name Things Clearly

Use names that reveal intent without extra explanation. If a comment is needed only to explain an identifier, rename the identifier first.

## Prefer

- Domain language that matches the problem space.
- Specific nouns for values and data structures.
- Verb and noun pairs for functions.
- Boolean names that read like questions.

## Avoid

- Generic placeholders such as `data`, `value`, `item`, or `handler` when a better name exists.
- Names that encode implementation details instead of meaning.
- Abbreviations that are not already standard in the codebase.
- Long names that repeat surrounding context unnecessarily.

## Conventions

| Element | Preferred Form | Example |
| --- | --- | --- |
| Variables | Reveal intent | `userCount` not `n` |
| Functions | Verb plus noun | `getUserById` not `user` |
| Booleans | Question form | `isActive`, `hasPermission`, `canEdit` |
| Constants | SCREAMING_SNAKE_CASE when the codebase uses it | `MAX_RETRY_COUNT` |

## Review Checklist

- Would a new reader know what this identifier represents?
- Does the name describe purpose instead of implementation trivia?
- Is the name consistent with surrounding code and domain vocabulary?
- Could two similar names be confused at a glance?

Prefer renaming over commenting around a weak name.
