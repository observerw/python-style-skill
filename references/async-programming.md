# Async Programming with anyio

## Core Principles

- **Always use [`anyio`](https://anyio.readthedocs.io/en/stable/)** for async code
- **Structured Concurrency**: NEVER use `asyncio.create_task()` or `asyncio.gather()` — use `anyio.create_task_group()` to ensure all tasks are properly joined or cancelled
- **Parallel by Default**: When tasks CAN run in parallel, they MUST run in parallel using task groups

## Task Groups (MANDATORY for Parallelism)

**Rule**: Use `anyio.create_task_group()` for ALL concurrent execution. Task groups ensure that:
- All tasks complete before the context manager exits
- If one task fails, all others are automatically cancelled
- No "zombie tasks" are left running after errors

```python
# ❌ ANTI-PATTERN: Orphaned tasks, errors swallowed
task = asyncio.create_task(background_work())
results = await asyncio.gather(task_a(), task_b())

# ✅ BEST PRACTICE: Structured concurrency
async with anyio.create_task_group() as tg:
    tg.start_soon(background_work)
    tg.start_soon(task_a)
    tg.start_soon(task_b)
# All tasks guaranteed to be done/cancelled here
```

## Task Group Methods

- **`tg.start_soon(func, *args)`**: Fire-and-forget concurrent execution (no await)
- **`await tg.start(func, *args)`**: Wait for task initialization (task must call `task_status.started()`)

```python
async def my_service(*, task_status=anyio.TASK_STATUS_IGNORED):
    # Setup logic (e.g., bind socket)
    task_status.started("Ready")
    # Main loop
    await anyio.sleep_forever()

async with anyio.create_task_group() as tg:
    status = await tg.start(my_service)  # Waits until started() is called
    print(f"Service is {status}")
```

## Exception Handling

**Rule**: Task group exceptions require special handling via `ExceptionGroup`

```python
# Python 3.11+ (PREFERRED)
try:
    async with anyio.create_task_group() as tg:
        tg.start_soon(task_a)
        tg.start_soon(task_b)
except* ValueError as eg:
    for exc in eg.exceptions:
        log.error(f"Value error in group: {exc}")

# Python 3.10 and below
try:
    async with anyio.create_task_group() as tg:
        tg.start_soon(task_a)
        tg.start_soon(task_b)
except ExceptionGroup as eg:
    for exc in eg.exceptions:
        log.error(f"Error in group: {exc}")
```

## Cancellation and Timeouts

**Rule**: Use `anyio.fail_after()` for timeouts and `tg.cancel_scope` for explicit cancellation

```python
# Timeout for entire task group
try:
    with anyio.fail_after(5):  # 5 second limit
        async with anyio.create_task_group() as tg:
            tg.start_soon(fetch_api_1)
            tg.start_soon(fetch_api_2)
except TimeoutError:
    log.error("Parallel operations timed out")

# Explicit cancellation
async with anyio.create_task_group() as tg:
    tg.start_soon(long_running_task)
    if condition:
        tg.cancel_scope.cancel()  # Cancel all tasks in this group

# Shield critical cleanup from cancellation
with anyio.CancelScope(shield=True):
    await db.close()  # Won't be interrupted
```

## Collecting Results from Parallel Tasks

**Rule**: Use shared data structures to collect results (task groups don't return values directly)

```python
# ✅ Collect results via shared list
results = []

async def fetch_and_store(url: str) -> None:
    data = await fetch(url)
    results.append(data)

async with anyio.create_task_group() as tg:
    for url in urls:
        tg.start_soon(fetch_and_store, url)
# results now contains all data
```

## Common Anti-Patterns to Avoid

- ❌ **Awaiting `start_soon()`**: It returns `None` and is not awaitable
- ❌ **Using task group outside context manager**: Invalid after `async with` block exits
- ❌ **Catching `Exception` instead of `ExceptionGroup`**: Won't properly handle multiple task failures
- ❌ **Infinite loops without `await`**: Must have at least one `await` (e.g., `anyio.sleep(0)`) to yield control
- ❌ **Using `asyncio.gather()` or `asyncio.create_task()`**: Breaks structured concurrency guarantees

## Why Task Groups > `asyncio.gather()`

| Feature | `asyncio.gather()` | `anyio.TaskGroup` |
| :--- | :--- | :--- |
| **Error Handling** | If one fails, others keep running (leaking resources) | If one fails, all others are **automatically cancelled** |
| **Lifecycle** | Return values are easy, cleanup is hard | Cleanup is guaranteed, results require manual collection |
| **Nesting** | Flat execution | Naturally hierarchical (Structured) |

**Zombie Task Pitfall**: If `gather(a(), b())` is called and `a()` raises an exception, `b()` will continue to run in the background. In a Task Group, `b()` is immediately cancelled, ensuring system state remains consistent.
