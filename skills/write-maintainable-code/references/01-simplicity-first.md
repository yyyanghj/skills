# Prefer Simplicity First

Choose the smallest design that is obviously correct for the current requirement. Optimize for local reasoning: a reader should understand the code in one pass without reconstructing a framework in their head.

Prefer boring code over impressive code. Future changes are easier when the current code is obvious.

## Use This Rule When

- Starting a new function, module, or API.
- Choosing between inline logic and a reusable abstraction.
- Deciding whether to add configuration, inheritance, or a generic helper.

## Prefer

- One clear flow from input to output.
- Existing language features before custom frameworks.
- Concrete types and straightforward data shapes.
- Short-lived duplication when the shared shape is not stable yet.

## Avoid

- Abstract base types used only once.
- Extension points for features that do not exist yet.
- Indirection added only to look scalable.
- Optimizations without a measured bottleneck.

## Ask Before Adding Complexity

- Does this solve a real requirement that exists today?
- Will at least two current call sites benefit from the abstraction?
- Does the abstraction make the happy path easier to understand?
- Would removing this layer make the code clearer? If yes, remove it.

## Review Checklist

- Can a new reader explain the code after one read?
- Are the main data shapes visible at the point of use?
- Are extra interfaces, classes, or helpers paying for themselves right now?
- Is the simplest version already good enough?

## Example

**Bad:**

```ts
interface IValidatorRule {
  validate(input: UserInput): string | null;
}

class RequiredFieldRule implements IValidatorRule {
  validate(input: UserInput): string | null {
    if (!input.username || input.username.trim() === "") {
      return "Username is required";
    }
    return null;
  }
}

class PasswordLengthRule implements IValidatorRule {
  validate(input: UserInput): string | null {
    if (!input.password || input.password.length < 8) {
      return "Password must be at least 8 chars";
    }
    return null;
  }
}

class ValidationEngine {
  private rules: IValidatorRule[];

  constructor(rules: IValidatorRule[]) {
    this.rules = rules;
  }

  run(input: UserInput): string[] {
    const errors: string[] = [];

    for (const rule of this.rules) {
      const error = rule.validate(input);
      if (error) errors.push(error);
    }

    return errors;
  }
}

const engine = new ValidationEngine([
  new RequiredFieldRule(),
  new PasswordLengthRule(),
]);
const errors = engine.run(userFormData);
```

**Good:**

```ts
function validateUser(input: UserInput): string[] {
  const errors: string[] = [];

  if (!input.username || input.username.trim() === "") {
    errors.push("Username is required");
  }

  if (!input.password || input.password.length < 8) {
    errors.push("Password must be at least 8 chars");
  }

  return errors;
}

const errors = validateUser(userFormData);
```
