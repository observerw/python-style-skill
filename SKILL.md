---
name: python-style
description: Check and fix Python code style, formatting, and linting errors. Enforces strict type hints and robust error handling. Use when the user asks to "fix lint errors", "format code", "check style", "improve python code quality".
---

# Python Style & Quality Standards

## Setup: Ruff Configuration

Before checking or fixing code style, ensure the project has Ruff configuration:

1. Check for existing configuration:
   - Look for `ruff.toml` or `.ruff.toml` in the project root
   - Look for `[tool.ruff]` section in `pyproject.toml`
2. If no configuration exists:
   - Preferred: Add the configuration from `assets/ruff.toml` to the project's `pyproject.toml` under `[tool.ruff]` section
   - Alternative: Copy `assets/ruff.toml` to the project root as `ruff.toml`
   - Always inform the user about the added configuration
3. Run Ruff commands: Always run check with autofix: `ruff check --fix <path>`

## Error Handling

- Domain-Specific Catching: Catch ONLY business/domain-level exceptions. Let all other exceptions CRASH the local context.
- Log at Entrypoints: Use top-level interface/entrypoint logging to capture and record unhandled crashes; do not litter code with defensive `try...except` blocks.

## Code Clarity & Static Analysis

- Static Access: MANDATORY static attribute access. NEVER use `getattr`/`setattr`/`hasattr` unless the logic is inherently dynamic and cannot be expressed otherwise.
- Composition Over Inheritance: Avoid deep inheritance or complex `__getattr__` that confuses static analysis.
- Import Rules:
  - Within the same module: Use relative imports (e.g., `from .module import func`).
  - Across different modules: Use absolute imports.
- Pattern Matching: Use `match` instead of multiple `if isinstance()` checks for better clarity.

## Type Hinting

- Advanced Type Practices:
  - Specific Types: Use specific types instead of `Any`.
  - Structured Returns: Use `TypedDict` or `NamedTuple` instead of plain `dict` or `tuple` for complex return values.
  - Protocols for Callables: Use a `Protocol` with a `__call__` method instead of `Callable`.
  - No Suppressions: Fix type errors instead of using `# type: ignore` or `cast(Any, ...)`.
  - Type Aliases: Use type aliases for complex or repeated types instead of redundant definitions.
- use `if TYPE_CHECKING` only if necessary. Do not abuse it.
- See [cheatsheet](references/type-hints-cheat-sheet.md) for 3.12+ type syntax.

## Functional Correctness & Safety

- Keyword-Only Defaults: Force parameters with defaults to be keyword-only by placing them after `*`.
