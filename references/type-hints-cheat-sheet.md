# Type Hints Cheat Sheet (Python 3.12)

## Type Parameters (PEP 695)

```python
# Generic functions
def func[T](x: T) -> T:
    return x

# Generic classes
class Stack[T]:
    def __init__(self) -> None:
        self.items: list[T] = []

    def push(self, item: T) -> None:
        self.items.append(item)

    def pop(self) -> T:
        return self.items.pop()

# Multiple type parameters
def pair[T, U](first: T, second: U) -> tuple[T, U]:
    return (first, second)

# Type aliases with parameters
type Vector[T] = list[T]
type Mapping[K, V] = dict[K, V]

# Bounded type parameters
def max[T: int | float](values: list[T]) -> T:
    return max(values)

# Default type parameters
type OptionalList[T = int] = list[T] | None
```

## Override Decorator (PEP 698)

```python
from typing import override

class Base:
    def method(self) -> int:
        return 42

class Derived(Base):
    @override
    def method(self) -> int:
        return 100
```

## Combined with Python 3.9+ Features

```python
# Built-in collection generics (Python 3.9+)
x: list[int] = [1]
x: dict[str, float] = {"key": 1.0}
x: tuple[int, str] = (1, "test")

# Union operator (Python 3.10+)
x: int | str = 42
x: int | None = None

from __future__ import annotations  # Lazy evaluation
```
