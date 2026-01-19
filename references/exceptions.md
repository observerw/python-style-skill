# Exception Design Guidelines

## Library vs Application

- Libraries:
  - MUST define a root base exception inheriting from `Exception`.
  - All public custom exceptions must inherit from this base.
  - Wrap 3rd-party/system exceptions to prevent implementation leakage (abstraction leaks).

- Applications:
  - Focus on semantic exceptions (e.g., `InsufficientFunds`, `UserNotFound`) that map to business logic failures.
  - Handle these at the interface layer (CLI/API boundary) to produce clean, user-friendly error reports.

## Granularity

- When to Create New Classes:
  - Introduce new exception classes only if catching them separately allows for different recovery strategies or distinct reporting logic.
- When to Reuse:
  - If the handling logic is the same, reuse existing types with descriptive messages.

## Best Practices

- Chaining:
  - Always use `raise ... from e` when wrapping exceptions to preserve the stack trace and context.
- Developer Errors:
  - Prefer standard built-ins (`ValueError`, `TypeError`) for developer errors (invalid usage/arguments) rather than custom exceptions.
