---
name: python-style
description: Check and fix Python code style, formatting, and linting errors. Enforces strict type hints and robust error handling. Use when the user asks to "fix lint errors", "format code", "check style", "improve python code quality".
---

# Python Style & Quality Standards

## Error Handling

- Domain-Specific Catching: Catch ONLY business/domain-level exceptions. Let all other exceptions CRASH the local context.
- Exception Chaining: Always use `raise ... from e` when re-raising or wrapping exceptions to preserve the stack trace.
- Log at Entrypoints: Use top-level interface/entrypoint logging to capture and record unhandled crashes; do not litter code with defensive `try...except` blocks.
- No Silent Failures: NEVER use `except: pass` or catch `Exception` just to suppress it.

## Code Clarity & Static Analysis

- Static Access: MANDATORY static attribute access. NEVER use `getattr`/`setattr`/`hasattr` unless the logic is inherently dynamic and cannot be expressed otherwise.
- Composition Over Inheritance: Avoid deep inheritance or complex `__getattr__` that confuses static analysis.
- Top-Level Imports: Place imports at the top; avoid local imports unless resolving circular dependencies.
- Import Rules:
  - Within the same module: Use relative imports (e.g., `from .module import func`).
  - Across different modules: Use absolute imports.
- Pattern Matching: Use `match` instead of multiple `if isinstance()` checks for better clarity.
- Manage Complexity: Refactor and split overly complex functions into smaller components.

## Type Hinting

- Strict Hints: Explicitly type all parameters and return values.
  - Specific Types: Use specific types instead of `Any` whenever possible.
  - Structured Returns: Use `TypedDict` or `NamedTuple` instead of plain `dict` or `tuple` for complex return values.
  - Protocols for Callables: Use a `Protocol` with a `__call__` method instead of `Callable`.
  - No Suppressions: Fix type errors instead of using `# type: ignore` or `cast(Any, ...)`.
  - Type Aliases: Use type aliases for complex or repeated types instead of redundant definitions.
- use `if TYPE_CHECKING` only if necessary. Do not abuse it.
- See [cheatsheet](references/type-hints-cheat-sheet.md) for 3.12+ type syntax.

## Functional Correctness & Safety

- No Mutable Defaults: Default to `None`, not mutables like `[]` or `{}`; initialize inside the function.
- Keyword-Only Defaults: Force parameters with defaults to be keyword-only by placing them after `*`.
- Comprehensions: Use comprehensions for creating collections, loops for side effects.
- Pathlib: Use `pathlib.Path` instead of `os.path` for path operations.
