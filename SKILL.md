---
name: python-style
description: Check and fix Python code style, formatting, and linting errors. Enforces strict type hints and robust error handling. Use when the user asks to "fix lint errors", "format code", "check style", "improve python code quality".
---

# Python Style & Quality Standards

## Workflow

1. **Check existing configuration**: Look for `ruff.toml`, `.ruff.toml`, or `[tool.ruff]` in `pyproject.toml`
2. **Setup configuration**:
   - If no config exists: Copy `assets/ruff.toml` to project root as `ruff.toml`
   - If config exists in `pyproject.toml`: Create `ruff.toml` by merging existing settings with `assets/ruff.toml` (preserve project-specific rules like `target-version`, `known-first-party`, custom ignores)
   - If `ruff.toml` already exists: Enhance it with missing rules from `assets/ruff.toml`
3. **Adapt to project**: Review and modify rules based on project needs (see below)
4. **Run Ruff**: `ruff check --fix <path>` and `ruff format <path>`

## Adapting ruff.toml to Project

The `assets/ruff.toml` provides a strict baseline. Adjust based on project context:

| Scenario        | Recommended Changes                                                |
| --------------- | ------------------------------------------------------------------ |
| Python < 3.10   | Change `target-version` accordingly                                |
| CLI/scripts     | Consider ignoring `T20` (print statements)                         |
| Data science    | May relax `ANN` for notebooks, add `"*.ipynb"` to per-file-ignores |
| Django/Flask    | Add `DJ`/`ASYNC` rules, ignore `RUF012` for mutable class attrs    |
| Legacy codebase | Start with fewer rules, incrementally enable more                  |
| Library/SDK     | Keep strict; add `D` (pydocstyle) for public API docs              |

Always set `known-first-party` in `[lint.isort]` to the project's package name.

## Best Practices

These guidelines cover design decisions that linters cannot enforce:

### Error Handling

- **Domain-Specific Catching**: Catch ONLY business/domain-level exceptions. Let unexpected exceptions propagate.
- **Log at Entrypoints**: Handle unhandled exceptions at top-level interfaces; avoid defensive `try...except` blocks throughout code.

### Code Clarity

- **Static Access**: Avoid `getattr`/`setattr`/`hasattr` unless logic is inherently dynamic.
- **Composition Over Inheritance**: Prefer composition; avoid deep inheritance or magic `__getattr__`.
- **Pattern Matching**: Use `match` instead of chained `if isinstance()` checks.

### Type Hinting

- Use `TypedDict`/`NamedTuple` for complex return values instead of plain `dict`/`tuple`
- Use `Protocol` with `__call__` instead of `Callable` for complex signatures
- See [cheatsheet](references/type-hints-cheat-sheet.md) for 3.12+ syntax

### Function Design

- **Keyword-Only Defaults**: Place parameters with defaults after `*` to force keyword-only usage
