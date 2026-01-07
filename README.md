# Python Style & Quality Standards

An agent skill for checking and fixing Python code style, formatting, and linting errors with strict type hints and robust error handling practices.

## Overview

This skill provides comprehensive guidelines and automated tooling to enforce high-quality Python code standards. It emphasizes:

- **Type Safety**: Strict type hints with Python 3.12+ syntax
- **Error Handling**: Domain-specific exception handling without silent failures
- **Code Clarity**: Static analysis-friendly code patterns
- **Functional Correctness**: Safe defaults and modern Python idioms

## Usage

This skill is automatically invoked when you ask to:

- "fix lint errors"
- "format code"
- "check style"
- "improve python code quality"

## Core Principles

### 1. Error Handling

- **Domain-Specific Catching**: Only catch business/domain-level exceptions; let others crash
- **Exception Chaining**: Always use `raise ... from e` to preserve stack traces
- **Log at Entrypoints**: Capture crashes at top-level interfaces, not throughout code
- **No Silent Failures**: Never use `except: pass` or suppress exceptions

```python
# ✅ Good
try:
    result = process_payment(amount)
except PaymentError as e:
    raise InsufficientFundsError("Payment failed") from e

# ❌ Bad
try:
    result = process_payment(amount)
except Exception:
    pass
```

### 2. Type Hinting

- **Strict Hints**: Type all parameters and return values
- **Specific Types**: Avoid `Any` whenever possible
- **Structured Returns**: Use `TypedDict` or `NamedTuple` over plain `dict`/`tuple`
- **Protocols over Callables**: Use `Protocol` with `__call__` method
- **No Suppressions**: Fix type errors, don't ignore them

```python
# ✅ Good - Python 3.12+ syntax
def process[T](items: list[T]) -> dict[str, T]:
    return {str(i): item for i, item in enumerate(items)}

# ✅ Good - Structured return
from typing import TypedDict

class UserData(TypedDict):
    name: str
    age: int

def get_user() -> UserData:
    return {"name": "Alice", "age": 30}
```

See [Type Hints Cheat Sheet](references/type-hints-cheat-sheet.md) for Python 3.12+ syntax examples.

### 3. Code Clarity & Static Analysis

- **Static Access**: Use direct attribute access; avoid `getattr`/`setattr`/`hasattr`
- **Composition Over Inheritance**: Prefer composition to complex inheritance
- **Top-Level Imports**: Keep imports at the top unless resolving circular dependencies
- **Pattern Matching**: Use `match` instead of multiple `isinstance()` checks
- **Manage Complexity**: Split complex functions into smaller components

```python
# ✅ Good
match value:
    case int(x):
        return x * 2
    case str(s):
        return s.upper()
    case _:
        return None

# ❌ Bad
if isinstance(value, int):
    return value * 2
elif isinstance(value, str):
    return value.upper()
else:
    return None
```

### 4. Functional Correctness & Safety

- **No Mutable Defaults**: Use `None` and initialize inside function
- **Keyword-Only Defaults**: Place parameters with defaults after `*`
- **Comprehensions**: Use for creating collections; loops for side effects
- **Pathlib**: Use `pathlib.Path` instead of `os.path`

```python
# ✅ Good
def add_item(*, items: list[str] | None = None) -> list[str]:
    if items is None:
        items = []
    items.append("new")
    return items

# ❌ Bad
def add_item(items: list[str] = []) -> list[str]:
    items.append("new")
    return items
```

## Linting Configuration

This skill includes a Ruff configuration ([assets/ruff.toml](assets/ruff.toml)) that enforces:

- **Error/Warning Detection** (`E`, `F`)
- **Import Sorting** (`I`)
- **Bugbear Rules** (`B`)
- **Type Annotations** (`ANN`)
- **Blind Exception Handling** (`BLE`)
- **Upgrade Syntax** (`UP`)

Exceptions:

- `ANN101`, `ANN102` (self/cls annotations) are ignored

## License

MIT
