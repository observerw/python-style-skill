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

## Anti-Patterns (PROHIBITED)

### Pointless Exception Wrapping

**NEVER** catch broad exceptions only to re-raise as different exceptions without adding value:

```python
# ❌ GARBAGE CODE - DELETE THIS
try:
    some_operation()
except Exception:
    raise CustomError()  # No context added, no recovery logic

# ❌ GARBAGE CODE - Still worthless even with message
try:
    some_operation()
except Exception as e:
    raise CustomError("Operation failed")  # Generic message adds nothing

# ❌ GARBAGE CODE - Just re-raising with different type
try:
    some_operation()
except Exception as e:
    raise CustomError(str(e))  # Pointless type conversion
```

**Exception wrapping is ONLY justified when:**

1. **Adding meaningful context/data (HIGH BAR - must pass all checks):**
   - ✅ Must add NEW information not in original exception (e.g., user_id, file_path, request_id)
   - ✅ Must be at a layer where this context is naturally available
   - ❌ Generic messages like "Operation failed" / "Error occurred" are NOT meaningful
   - ❌ Just reformatting the original message is NOT meaningful
```python
# ✅ ACCEPTABLE - Adding concrete domain context
try:
    user_data = fetch_user(user_id)
except HTTPError as e:
    raise UserFetchError(
        f"Failed to fetch user {user_id} from endpoint {endpoint}",
        user_id=user_id,
        endpoint=endpoint
    ) from e

# ❌ STILL GARBAGE - No new information
try:
    user_data = fetch_user(user_id)
except HTTPError as e:
    raise UserFetchError("Failed to fetch user") from e  # user_id already in traceback!
```

2. **Converting at API boundaries (LIBRARIES ONLY - strict criteria):**
   - ✅ ONLY in libraries exposing public APIs
   - ✅ ONLY to hide implementation details from library users
   - ✅ Must document this wrapping in library's exception hierarchy
   - ❌ NEVER in application code (applications should use semantic exceptions directly)
   - ❌ NEVER just to "standardize" exception types
```python
# ✅ ACCEPTABLE - Library hiding SQLAlchemy implementation
# (documented in MyLibrary's exception hierarchy)
try:
    result = self._db_session.query(Model).first()
except SQLAlchemyError as e:
    raise MyLibraryDatabaseError("Database operation failed") from e

# ❌ GARBAGE - Application code wrapping
try:
    user = db.get_user(id)  # Already returns domain exceptions
except DatabaseError as e:
    raise UserServiceError("Failed") from e  # Pointless layer
```

3. **Implementing recovery logic (MUST actually recover):**
   - ✅ Must have CONCRETE fallback/retry/default behavior
   - ✅ Must NOT re-raise after "recovering" (that's not recovery!)
   - ❌ Just logging is NOT recovery (use a logger at the call site instead)
   - ❌ Re-raising a different exception is NOT recovery
```python
# ✅ ACCEPTABLE - Actual recovery with fallback
try:
    data = cache.get(key)
except CacheConnectionError as e:
    logger.warning("Cache unavailable, using database fallback", exc_info=e)
    data = database.get(key)  # Real fallback!
    return data

# ❌ GARBAGE - "Logging and re-raising" is NOT recovery
try:
    data = cache.get(key)
except CacheConnectionError as e:
    logger.error("Cache failed")  # Logging doesn't make this recovery!
    raise ServiceError("Cache unavailable") from e  # Still propagating = not recovered
```

**Default action:** If unsure, DELETE the try-except block and let the original exception propagate.

**Decision checklist before wrapping any exception:**
1. Am I adding NEW data that isn't already in the exception/traceback? (not just rewording)
2. Is this a library's public API boundary? (if application code, answer is NO)
3. Am I implementing REAL recovery that returns a value without re-raising?

If you answered NO to all three questions above, DELETE the exception wrapper. The original exception is better.
