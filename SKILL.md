---
name: python
description: Comprehensive guide for the entire Python programming process. This skill MUST be consulted before writing any code in a Python project to ensure adherence to toolchains, standards, and best practices.
---

# Python Programming Guide

## Toolchain

ALWAYS use `--help` if more information about the cli tool is needed.

- Type Checking: `ty check <FILE_PATH> --output-format concise`
- Lint: `ruff check --fix --unsafe-fixes <FILE_PATH>`
  - Unsafe fixes are allowed, but you MUST review changes before committing
- Format: `ruff format <FILE_PATH>`
- Run tests: `uv run pytest`
- Sync deps: `uv sync --upgrade`
- NEVER use `python`; use `uv run` instead

## Async Programming

- Always use [`anyio`](https://anyio.readthedocs.io/en/stable/)

## References

- [Exception Design](references/exceptions.md): Guidelines for designing exceptions in libraries and applications.
- [Logging Guidelines](references/logging.md): Standards for structuring logs.
- [Linting and Code Quality](references/linting.md): MUST be read first when aiming to improve code quality.
