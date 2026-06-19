# Python Coding Style Guidelines

A comprehensive guide rooted in the Zen of Python (PEP 20), official PEPs, and
established software engineering literature — notably *Design Patterns* (Gamma
et al., 1994) and *Clean Code* (Martin, 2008).

> **Python 3 only.** This guide targets **Python 3.9+** (3.11+ recommended).

---

## Table of Contents

1. [Philosophy](#1-philosophy)
2. [Code Layout & Formatting (PEP 8)](#2-code-layout--formatting-pep-8)
3. [Naming Conventions](#3-naming-conventions)
4. [Functions & Methods](#4-functions--methods)
5. [Classes & Object-Oriented Design](#5-classes--object-oriented-design)
6. [Documentation & Docstrings (PEP 257)](#6-documentation--docstrings-pep-257)
7. [Type Hints (PEP 484 / 526 / 604)](#7-type-hints-pep-484--526--604)
8. [Error Handling](#8-error-handling)
9. [Imports (PEP 328)](#9-imports-pep-328)
10. [Design Patterns in Python](#10-design-patterns-in-python)
11. [Testing](#11-testing)
12. [Database Access & ACID](#12-database-access--acid)
13. [Concurrency & Async (PEP 3156)](#13-concurrency--async-pep-3156)
14. [Performance & Pythonic Idioms](#14-performance--pythonic-idioms)
15. [Defensive Programming & Input Validation](#15-defensive-programming--input-validation)
16. [Project Structure & Packaging (PEP 517 / 621)](#16-project-structure--packaging-pep-517--621)
17. [Tooling](#17-tooling)
18. [Build Tools](#18-build-tools)
19. [SBOM Creation](#19-sbom-creation)
20. [References](#20-references)

---

## 1. Philosophy

### 1.1 The Zen of Python (PEP 20)

Every design decision should be weighed against these aphorisms — run
`import this` to print them:

> Beautiful is better than ugly.
> Explicit is better than implicit.
> Simple is better than complex.
> Complex is better than complicated.
> Flat is better than nested.
> Sparse is better than dense.
> Readability counts.
> Special cases aren't special enough to break the rules.
> Although practicality beats purity.
> Errors should never pass silently.
> Unless explicitly silenced.
> In the face of ambiguity, refuse the temptation to guess.
> There should be one — and preferably only one — obvious way to do it.
> Although that way may not be obvious at first unless you're Dutch.
> Now is better than never.
> Although never is often better than *right* now.
> If the implementation is hard to explain, it's a bad idea.
> If the implementation is easy to explain, it may be a good idea.
> Namespaces are one honking great idea — let's do more of those!

### 1.2 Guiding Principles

| Principle | Source | Summary |
|---|---|---|
| **Readability over cleverness** | PEP 20, *Clean Code* Ch. 1 | Code is read far more than it is written. Optimise for the reader. |
| **Single Responsibility** | *Clean Code* Ch. 10, SOLID | Every module, class, and function should have one reason to change. |
| **Don't Repeat Yourself (DRY)** | *The Pragmatic Programmer* | Extract duplication into a single, authoritative source of truth. |
| **YAGNI** | XP / *Clean Code* | Do not build for hypothetical future requirements. |
| **Least Surprise** | PEP 20, general SE | APIs should behave the way a reasonable user would expect. |
| **Favour Composition** | *Design Patterns* Ch. 1 | Prefer composition over inheritance; inherit interfaces, compose behaviour. |
| **Open/Closed** | SOLID | Software entities should be open for extension but closed for modification. |

---

## 2. Code Layout & Formatting (PEP 8)

PEP 8 is the authoritative style guide for Python. The rules below summarise
its key points and add practical clarifications.

### 2.1 Indentation

- Use **4 spaces** per indentation level. Never mix tabs and spaces.
- Continuation lines should align with the opening delimiter or use a hanging
  indent with an extra level of indentation:

```python
# Aligned with opening delimiter
result = some_function(arg_one, arg_two,
                       arg_three, arg_four)

# Hanging indent — add an extra level to distinguish from the body
def long_function_name(
        var_one, var_two,
        var_three, var_four):
    print(var_one)
```

### 2.2 Maximum Line Length

- **79 characters** for code, **72 characters** for docstrings and comments
  (PEP 8).
- Teams may agree on **88** (Black default) or **99** as an upper bound, but
  the limit must be enforced consistently by a formatter.

### 2.3 Blank Lines (PEP 8)

- **Two blank lines** before and after every top-level function and class
  definition.
- **One blank line** between method definitions inside a class.
- **One blank line** after the class docstring, before the first method.
- **No blank line** after a `def` line or after an `if`/`for`/`while`/`with`
  colon line.
- Use **single blank lines** sparingly within functions to separate logical
  sections.
- **No trailing blank lines** at the end of a file (a single newline
  terminator is required).

### 2.4 Whitespace

```python
# Yes
spam(ham[1], {eggs: 2})
x = 1
long_variable = 3
d = {"key": value}

# No
spam( ham[ 1 ], { eggs: 2 } )
x             = 1
long_variable = 3
d = {"key" :value}
```

### 2.5 Trailing Commas

Use trailing commas in multi-line structures. They produce cleaner diffs and
reduce merge conflicts:

```python
FRUITS = [
    "apple",
    "banana",
    "cherry",
]
```

### 2.6 Shebang Line

Every Python file that is intended to be executed directly **must** start with
a shebang line. Use the `env` form for portability:

```python
#!/usr/bin/env python3
```

- Place the shebang on the very first line — before encoding declarations,
  docstrings, and imports.
- Library modules that are only ever imported (never run directly) may omit
  the shebang.

### 2.7 The `if __name__ == "__main__"` Guard

Every script **must** wrap its entry-point logic in a main guard. This ensures
the file can be both executed directly and imported without side effects:

```python
#!/usr/bin/env python3
"""Nightly report generator."""

import sys


def generate_report(date: str) -> None:
    """Generate and email the nightly report.

    :param date: The report date in ISO 8601 format (``YYYY-MM-DD``).
    :type date: str
    """
    ...


def main() -> None:
    """Parse arguments and run the report."""
    if len(sys.argv) != 2:
        sys.exit("Usage: generate_report.py <date>")
    generate_report(sys.argv[1])


if __name__ == "__main__":
    main()
```

**Why both matter together:**

- The **shebang** lets the OS find the interpreter (`./my_script.py` instead of
  `python3 my_script.py`).
- The **main guard** prevents top-level code from running on `import`, which
  would break testing, tooling, and reuse.
- Extract the actual work into a `main()` function so it can be called from
  tests or other modules without invoking `sys.argv` parsing.

---

## 3. Naming Conventions

| Entity | Convention | Example |
|---|---|---|
| Module | `snake_case` | `data_loader.py` |
| Package | `short, all-lowercase` | `utils`, `models` |
| Class | `PascalCase` | `HttpClient`, `UserRepository` |
| Exception | `PascalCase` ending in `Error` | `ValidationError` |
| Function / Method | `snake_case` | `calculate_total()` |
| Variable | `snake_case` | `total_count` |
| Constant | `UPPER_SNAKE_CASE` | `MAX_RETRIES` |
| Type Variable | `PascalCase`, short | `T`, `KT`, `VT` |
| Private | Single leading underscore | `_internal_helper()` |
| Name-mangled | Double leading underscore | `__private_attr` |
| Dunder / Magic | Double leading + trailing | `__init__`, `__repr__` |

### 3.1 Naming Guidance (*Clean Code* Ch. 2)

- **Use intention-revealing names.** `elapsed_time_in_days` beats `d`.
- **Avoid disinformation.** Don't name a list `account_list` if it is not
  actually a `list` — use `accounts`.
- **Make meaningful distinctions.** `get_active_account()` vs
  `get_account()` — not `get_account2()`.
- **Use pronounceable, searchable names.** Grep-ability matters.
- **Avoid encodings.** No Hungarian notation (`str_name`), no member prefixes
  (`m_value`). Python's dynamic nature and type hints make them redundant.

---

## 4. Functions & Methods

### 4.1 Size and Focus (*Clean Code* Ch. 3)

- Functions should be **small** — ideally under 20 lines.
- Each function does **one thing** at **one level of abstraction**.
- Extract nested logic into well-named helper functions rather than adding
  depth.

### 4.2 Arguments

- **Fewer arguments are better.** Zero (niladic) is ideal; three (triadic)
  should be rare. Beyond three, use a dataclass or a typed dict.
- **No flag arguments** — a boolean that switches behaviour means the function
  does two things. Split it into two functions.
- Use **keyword-only arguments** (PEP 3102) for clarity when a function has
  more than two parameters:

```python
def connect(host: str, port: int, *, timeout: float = 30.0, retries: int = 3):
    ...
```

### 4.3 Return Values

- Prefer returning **values** over mutating arguments.
- A function should return a **consistent type**. Avoid returning `None` as a
  sentinel mixed with real values — raise an exception or use the Null Object
  pattern instead.
- For functions that may or may not find a result, return `T | None` and
  annotate explicitly rather than raising on every miss.

### 4.4 Side Effects

- Clearly distinguish **commands** (perform an action, return nothing) from
  **queries** (return a value, cause no side effects) — the Command-Query
  Separation principle (*Clean Code* Ch. 3).
- If a function has side effects, make them obvious from the name:
  `save_to_disk()`, `send_notification()`.

### 4.5 Avoid Lambdas — Prefer Named Functions

Lambdas are anonymous, untestable, and invisible in tracebacks. **Avoid them**
in favour of named functions or named callable objects. The only acceptable use
is a trivial key function in a call that would be less readable with a
separate `def`.

```python
# Acceptable — trivial sort key, clearer inline
users.sort(key=lambda u: u.last_name)

# Bad — non-trivial logic hidden in a lambda
transform = lambda x: x.strip().lower().replace("-", "_")

# Good — named, testable, shows up in tracebacks
def normalise_key(x: str) -> str:
    """Normalise a string for use as a dictionary key.

    :param x: The raw input string.
    :type x: str
    :returns: A cleaned, lowercased, underscore-separated string.
    :rtype: str
    """
    return x.strip().lower().replace("-", "_")

transform = normalise_key
```

**Why not lambdas:**

- **PEP 8** explicitly discourages assigning a lambda to a variable — use
  `def` instead.
- Lambdas have **no name** in tracebacks, making debugging harder.
- Lambdas **cannot contain statements** (assignments, `raise`, `return`),
  leading to contorted expressions when logic grows.
- Lambdas **cannot have docstrings** or type annotations.
- Named functions can be **unit-tested directly**; lambdas cannot.

Use `operator` module functions as drop-in replacements for common lambdas:

```python
from operator import attrgetter, itemgetter, methodcaller

# Instead of: lambda u: u.age
users.sort(key=attrgetter("age"))

# Instead of: lambda d: d["name"]
records.sort(key=itemgetter("name"))

# Instead of: lambda s: s.upper()
results = list(map(methodcaller("upper"), names))
```

When passing a callable to `map()`, `filter()`, or `functools.reduce()`,
prefer a named function or a comprehension:

```python
# Prefer comprehension over map + lambda
squares = [x ** 2 for x in numbers]

# If map is used, pass a named function
results = list(map(normalise_key, raw_keys))
```

---

## 5. Classes & Object-Oriented Design

### 5.1 Class Design (*Clean Code* Ch. 6 & 10)

- Classes should be **small** — measured not in lines but in
  **responsibilities**.
- Aim for **high cohesion**: every method should use most of the instance
  variables.
- Follow the **Single Responsibility Principle (SRP)**: a class has one reason
  to change.

### 5.2 Encapsulation

- Default to making attributes and methods **private** (single underscore `_`).
  Expose only what clients need.
- Use `@property` for computed attributes or to enforce invariants:

```python
class Temperature:
    """Represent a temperature with Celsius as the canonical unit.

    :param celsius: The temperature in degrees Celsius.
    :type celsius: float
    """

    def __init__(self, celsius: float):
        self._celsius = celsius

    @property
    def fahrenheit(self) -> float:
        """Return the temperature converted to Fahrenheit.

        :returns: Temperature in degrees Fahrenheit.
        :rtype: float
        """
        return self._celsius * 9 / 5 + 32
```

### 5.3 Inheritance vs. Composition (*Design Patterns* Ch. 1)

> "Favour object composition over class inheritance."

- Use inheritance for **is-a** relationships and when you control the base
  class.
- Use **composition** (has-a) or **dependency injection** for everything else.
- Prefer **Abstract Base Classes** (`abc.ABC`) or **Protocols** (PEP 544) to
  define interfaces rather than deep inheritance trees.

```python
import json
from typing import Any, Protocol

class Serializer(Protocol):
    """Interface for objects that serialize structured data to bytes."""

    def serialize(self, data: dict[str, Any]) -> bytes:
        """Serialize a mapping to bytes.

        :param data: The data to serialize.
        :type data: dict[str, Any]
        :returns: The serialized byte string.
        :rtype: bytes
        """
        ...

class JsonSerializer:
    """Serialize data to JSON-encoded UTF-8 bytes."""

    def serialize(self, data: dict[str, Any]) -> bytes:
        """Serialize a mapping to JSON bytes.

        :param data: The data to serialize.
        :type data: dict[str, Any]
        :returns: JSON-encoded UTF-8 byte string.
        :rtype: bytes
        """
        return json.dumps(data).encode()

class MessageBroker:
    """Publish messages using a pluggable serialization strategy.

    :param serializer: The serializer used to encode messages.
    :type serializer: Serializer
    """

    def __init__(self, serializer: Serializer):
        self._serializer = serializer
```

### 5.4 Data Classes (PEP 557)

Use `@dataclass` for classes that are primarily data containers. They
auto-generate `__init__`, `__repr__`, `__eq__`, and more:

```python
from dataclasses import dataclass, field

@dataclass(frozen=True)
class Coordinate:
    """An immutable geographic coordinate.

    :param latitude: Degrees north (positive) or south (negative).
    :type latitude: float
    :param longitude: Degrees east (positive) or west (negative).
    :type longitude: float
    :param label: Optional human-readable label for the location.
    :type label: str
    """

    latitude: float
    longitude: float
    label: str = ""
```

- Use `frozen=True` to make instances immutable when possible.
- Use `__slots__` (or `slots=True` on Python 3.10+) for memory-critical paths.

### 5.5 SOLID Principles

| Principle | Python Application |
|---|---|
| **S** — Single Responsibility | One module or class per concern. |
| **O** — Open/Closed | Extend behaviour via Strategy, Decorator, or subclassing — not by editing existing code. |
| **L** — Liskov Substitution | Subtypes must be drop-in replacements; honour preconditions, postconditions, and invariants. |
| **I** — Interface Segregation | Define narrow `Protocol` types; don't force clients to depend on methods they don't use. |
| **D** — Dependency Inversion | Depend on abstractions (`Protocol`, `ABC`), not concretions. Inject dependencies through constructors. |

---

## 6. Documentation & Docstrings (PEP 257)

### 6.1 When to Write Docstrings

- **All public modules, classes, functions, and methods** must have docstrings.
- Private helpers generally do not need docstrings if the name and type hints
  are self-explanatory.

### 6.2 Docstring Format

Use **reStructuredText (rST)** style — the native format for Sphinx and the
most widely supported across Python tooling:

```python
def fetch_user(user_id: int, *, include_deleted: bool = False) -> User:
    """Retrieve a user by their primary key.

    Looks up the user in the cache first, then falls back to the database.

    :param user_id: The unique identifier for the user.
    :type user_id: int
    :param include_deleted: If True, soft-deleted users are also returned.
    :type include_deleted: bool
    :returns: The matching User object.
    :rtype: User
    :raises UserNotFoundError: If no user matches the given id.
    """
```

#### Field List Reference

| Directive | Purpose | Example |
|---|---|---|
| `:param name:` | Describe a parameter | `:param user_id: The unique identifier.` |
| `:type name:` | Declare a parameter's type | `:type user_id: int` |
| `:returns:` | Describe the return value | `:returns: The matching user.` |
| `:rtype:` | Declare the return type | `:rtype: User` |
| `:raises ExcType:` | Document a raised exception | `:raises ValueError: If n < 0.` |
| `:var name:` | Document a module/class variable | `:var MAX_RETRIES: Default retry limit.` |
| `:vartype name:` | Declare a variable's type | `:vartype MAX_RETRIES: int` |

#### Combining `:param:` and `:type:` on One Line

When brevity is preferred, the type can be inlined into the `:param:` directive:

```python
def connect(host: str, port: int, *, timeout: float = 30.0) -> Connection:
    """Open a TCP connection to the given host.

    :param str host: Hostname or IP address.
    :param int port: Port number.
    :param float timeout: Seconds to wait before aborting (default 30.0).
    :returns: An established connection.
    :rtype: Connection
    :raises ConnectionError: If the host is unreachable.
    """
```

#### Multiple Return Values

For functions that return tuples, document each element:

```python
def parse_address(raw: str) -> tuple[str, int]:
    """Parse a host:port string.

    :param raw: A string in the format ``"host:port"``.
    :type raw: str
    :returns: A tuple of ``(host, port)``.
    :rtype: tuple[str, int]
    :raises ValueError: If the string is not in the expected format.
    """
```

#### Class Docstrings

Document the class purpose at the class level and constructor parameters
under `__init__`:

```python
class RetryPolicy:
    """Control retry behaviour for failed operations.

    :var int DEFAULT_MAX: The default maximum number of retries.
    """

    DEFAULT_MAX: int = 3

    def __init__(self, max_retries: int = DEFAULT_MAX, backoff: float = 1.0):
        """Create a new retry policy.

        :param max_retries: Maximum number of retry attempts.
        :type max_retries: int
        :param backoff: Base delay in seconds between retries (doubled each attempt).
        :type backoff: float
        """
        self._max_retries = max_retries
        self._backoff = backoff
```

- **One-line docstrings** for obvious functions:

```python
def square(n: float) -> float:
    """Return the square of *n*."""
    return n * n
```

#### Docstring Conventions Summary

- Use triple double-quotes (`"""`) for all docstrings.
- First line is a **single-line summary** in imperative mood ("Return …", not
  "Returns …").
- Separate the summary from the body with a **blank line**.
- Place `:param:` / `:type:` blocks **before** `:returns:` / `:rtype:`, which
  come **before** `:raises:`.
- Use rST inline markup: single backticks for code (`` :param: ``), double
  backticks for literals (````"host:port"````), and `*italics*` for emphasis.

### 6.3 Comments (*Clean Code* Ch. 4)

- **Comments do not compensate for bad code.** If you need a comment to explain
  *what* the code does, rewrite the code.
- **Good comments** explain *why* — constraints, trade-offs, workarounds for
  external bugs.
- **Bad comments** are redundant, misleading, or mandated boilerplate. Remove
  them.
- Never commit commented-out code. Version control exists for a reason.

---

## 7. Type Hints (PEP 484 / 526 / 604)

### 7.1 General Rules

- **Annotate all public function signatures.** Internal helpers benefit from
  annotations too, but it is acceptable to omit them when types are obvious
  from context.
- Use **built-in generics** (Python 3.9+): `list[int]`, `dict[str, Any]`,
  `tuple[int, ...]` instead of importing from `typing`.
- Use the **union shorthand** (Python 3.10+): `int | None` instead of
  `Optional[int]`.
- This guide targets **Python 3.9+** (3.11+ recommended). Python 2 is end-of-life
  (sunset 2020-01-01); do not write new Python 2 code.

### 7.2 Common Patterns

```python
from collections.abc import Callable, Iterable, Mapping, Sequence

def process_items(
    items: Sequence[str],
    transform: Callable[[str], str] | None = None,
) -> list[str]:
    """Apply an optional transform to each item.

    :param items: The items to process.
    :type items: Sequence[str]
    :param transform: A function applied to each item, or ``None`` to
        pass items through unchanged.
    :type transform: Callable[[str], str] or None
    :returns: The processed items.
    :rtype: list[str]
    """
    ...

# Type alias statement (PEP 695, Python 3.12+) for complex types
type JsonValue = str | int | float | bool | None | list["JsonValue"] | dict[str, "JsonValue"]

# On Python 3.10–3.11, use the PEP 613 ``TypeAlias`` annotation instead:
# JsonValue: TypeAlias = str | int | float | bool | None | list["JsonValue"] | dict[str, "JsonValue"]
```

### 7.3 Generics (PEP 695)

Python 3.12+ provides a clean generic syntax:

```python
def first[T](items: Sequence[T]) -> T:
    """Return the first element of a non-empty sequence.

    :param items: A non-empty sequence.
    :type items: Sequence[T]
    :returns: The first element.
    :rtype: T
    """
    return items[0]

class Stack[T]:
    """A generic LIFO stack."""

    def __init__(self) -> None:
        self._items: list[T] = []

    def push(self, item: T) -> None:
        """Add an item to the top of the stack.

        :param item: The item to push.
        :type item: T
        """
        self._items.append(item)
```

### 7.4 Runtime vs. Static Typing

- Type hints are **not enforced at runtime** by default. Use a static type
  checker (mypy, pyright) in CI.
- For runtime validation at system boundaries, use **Pydantic** or
  `@beartype`.

### 7.5 Typed Data Passing

Pass **typed objects** between functions and modules — not raw `dict`s, loose
strings, or ad-hoc tuples. Typed data makes interfaces self-documenting,
catches errors at development time rather than runtime, and keeps refactoring
tractable.

- **Pass dataclasses, Pydantic models, or NamedTuples** instead of raw `dict`
  or `str` when data crosses function or module boundaries:

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class UserCreate:
    """Data required to register a new user.

    :param email: The user's email address.
    :type email: str
    :param display_name: The name shown in the UI.
    :type display_name: str
    """

    email: str
    display_name: str

# Yes — caller and callee agree on the shape at type-check time
def register_user(data: UserCreate) -> User:
    ...

# No — raw dict is opaque; typos and missing keys surface only at runtime
def register_user(data: dict) -> User:
    ...
```

- **Use type hints on all function signatures** to document expected data
  shapes. Type checkers and IDE autocomplete rely on these annotations.
- **At system boundaries** (API responses, config files, CLI arguments, environment
  variables), **parse into typed objects immediately**. Downstream code should
  never handle raw JSON dicts or unvalidated strings:

```python
from pydantic import BaseModel

class ApiResponse(BaseModel):
    """Typed representation of the external API response."""

    user_id: int
    status: str

response = ApiResponse.model_validate(raw_json)  # validate once, use typed data everywhere
```

- **Return typed objects**, not tuples of loosely related values. A
  `@dataclass` or `NamedTuple` is clearer than `tuple[str, int, bool]` and
  survives refactoring better.

---

## 8. Error Handling

### 8.1 Principles (PEP 20 + *Clean Code* Ch. 7)

> "Errors should never pass silently. Unless explicitly silenced."

- **Raise exceptions for exceptional conditions.** Do not use return codes.
- **Catch specific exceptions.** Never bare-`except` or `except Exception`
  without re-raising.
- **Fail fast.** Validate inputs at the boundary, then trust them downstream.

### 8.2 Exception Hierarchy

Define domain exceptions that inherit from a project-level base:

```python
class AppError(Exception):
    """Base class for all application-level errors."""

class NotFoundError(AppError):
    """Raised when a requested resource does not exist."""

class PermissionDeniedError(AppError):
    """Raised when the caller lacks required permissions."""
```

### 8.3 Context Managers

**Always** use context managers (`with` statements) for any resource that must
be acquired and released — files, sockets, database connections, locks, and
temporary state. Context managers guarantee cleanup even when exceptions occur,
eliminating an entire class of resource-leak bugs.

#### When to Use `with`

| Resource | Context Manager |
|---|---|
| Files | `open()` |
| Database connections / cursors | Driver-supplied or `contextlib.closing()` |
| Transactions | `session.begin()`, custom context manager |
| Locks | `threading.Lock()`, `asyncio.Lock()` |
| Temporary directories / files | `tempfile.TemporaryDirectory()`, `tempfile.NamedTemporaryFile()` |
| Decimal precision overrides | `decimal.localcontext()` |
| Suppressing exceptions | `contextlib.suppress(ExceptionType)` |
| Timing / profiling blocks | Custom context manager |
| Mocking / patching in tests | `unittest.mock.patch()` |

```python
# Yes — deterministic cleanup
with open("data.csv") as f:
    rows = f.readlines()

# No — leak if an exception occurs before close()
f = open("data.csv")
rows = f.readlines()
f.close()
```

#### Writing Custom Context Managers

For simple acquire/release pairs, use `contextlib.contextmanager`:

```python
from contextlib import contextmanager

@contextmanager
def database_transaction(conn):
    """Wrap a database operation in a transaction with automatic rollback.

    :param conn: An active database connection.
    :type conn: Connection
    :returns: A context manager yielding the active transaction.
    :rtype: Iterator[Transaction]
    :raises Exception: Re-raises any exception after rolling back.
    """
    tx = conn.begin()
    try:
        yield tx
        tx.commit()
    except Exception:
        tx.rollback()
        raise
```

For reusable or stateful managers, implement the protocol directly:

```python
class Timer:
    """Measure elapsed wall-clock time for a block of code.

    :ivar elapsed: Seconds elapsed after the block completes.
    :vartype elapsed: float
    """

    def __init__(self) -> None:
        self.elapsed: float = 0.0
        self._start: float = 0.0

    def __enter__(self) -> "Timer":
        self._start = time.perf_counter()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb) -> None:
        self.elapsed = time.perf_counter() - self._start
```

#### `contextlib` Utilities

The standard library provides powerful composable tools:

```python
from contextlib import suppress, closing, ExitStack

# Suppress a specific exception
with suppress(FileNotFoundError):
    os.remove("temp.dat")

# Wrap objects that have .close() but no __exit__
with closing(legacy_connection()) as conn:
    conn.execute(query)

# Manage a dynamic number of context managers
with ExitStack() as stack:
    files = [stack.enter_context(open(f)) for f in file_paths]
    process_all(files)
```

#### Async Context Managers

For async resources, use `async with` and `contextlib.asynccontextmanager`:

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def http_session():
    """Yield an HTTP session, closing it on exit.

    :returns: An async context manager yielding an aiohttp session.
    :rtype: AsyncIterator[aiohttp.ClientSession]
    """
    session = aiohttp.ClientSession()
    try:
        yield session
    finally:
        await session.close()
```

### 8.4 Exception Chaining (PEP 3134)

Always chain exceptions to preserve the original traceback:

```python
try:
    value = config["timeout"]
except KeyError as exc:
    raise ConfigurationError("Missing 'timeout' in config") from exc
```

### 8.5 Logging Over Printing

- Use the `logging` module, never `print()`, for operational output.
- Log at the appropriate level: `DEBUG` for diagnostics, `INFO` for key events,
  `WARNING` for recoverable issues, `ERROR` for failures.

---

## 9. Imports (PEP 328)

### 9.1 Ordering

Imports are grouped in this order, separated by a blank line:

1. **Standard library** (`os`, `sys`, `json`)
2. **Third-party** (`requests`, `sqlalchemy`, `pydantic`)
3. **Local / project** (`from myapp.models import User`)

Within each group, sort alphabetically. Use `isort` to automate this.

### 9.2 Style

```python
# Yes — absolute imports (PEP 328)
from myapp.services.auth import authenticate

# Acceptable — relative imports within a package
from .models import User

# No — wildcard imports pollute the namespace
from os.path import *
```

- Import **modules**, not individual names, when the module provides a coherent
  namespace: `import os.path` then `os.path.join(...)`.
- Import **specific names** when using them frequently and the source is
  unambiguous: `from datetime import datetime, timedelta`.
- Never use `import *` outside of `__init__.py` re-exports.

### 9.3 One Module per Concern — No Redundant Imports

When two or more modules can accomplish the same task, **pick one and use it
consistently** throughout the file (and ideally the entire project). Importing
multiple modules for overlapping functionality adds cognitive load, creates
subtle inconsistencies, and makes refactoring harder.

```python
# Bad — two modules for the same purpose in the same file
import time
from datetime import datetime

now_a = datetime.now()          # via datetime
now_b = time.time()             # via time — same concept, different module

# Good — choose one and stick with it
from datetime import datetime, timezone

now = datetime.now(tz=timezone.utc)
```

Common violations to watch for:

| Overlapping modules | Pick one |
|---|---|
| `time` / `datetime` for current time | `datetime` (higher-level, timezone-aware) |
| `os.path` / `pathlib` for path manipulation | `pathlib` (object-oriented, modern) |
| `json` / `simplejson` for serialisation | `json` (stdlib, unless you need a specific `simplejson` feature) |
| `urllib` / `requests` / `httpx` for HTTP | One HTTP library per project |
| `logging` / `print` for output | `logging` (never mix in production code) |
| `subprocess` / `os.system` for shell commands | `subprocess` (`os.system` is legacy) |
| `re` / `str` methods for simple matching | `str` methods when sufficient; `re` for true regex |
| `threading` / `multiprocessing` for the same task | Choose based on I/O-bound vs CPU-bound (see [Section 13](#13-concurrency--async-pep-3156)) |

**Rule of thumb:** if you catch yourself importing a second module that solves
a problem you have already solved with the first, delete the redundant import
and reuse the module you already depend on.

---

## 10. Design Patterns in Python

The Gang of Four patterns remain relevant, but Python's dynamic features often
simplify their implementation. Below are the most commonly applicable patterns
with Pythonic implementations.

### 10.1 Creational Patterns

#### Factory Method

Use a classmethod or a standalone function instead of a full factory class:

```python
class Notification:
    """Base class for user-facing notifications."""

    @classmethod
    def from_event(cls, event: Event) -> "Notification":
        """Create a notification appropriate to the event's severity.

        :param event: The domain event to build a notification from.
        :type event: Event
        :returns: An urgent or standard notification.
        :rtype: Notification
        """
        if event.severity == Severity.CRITICAL:
            return UrgentNotification(event.message)
        return StandardNotification(event.message)
```

#### Builder

Use keyword arguments, `@dataclass`, or a fluent interface:

```python
from dataclasses import dataclass, field, replace

@dataclass
class QueryBuilder:
    """Immutable builder for simple SQL SELECT statements.

    :param table: The table to query.
    :type table: str
    :param conditions: WHERE clause conditions.
    :type conditions: list[str]
    :param limit: Maximum number of rows to return.
    :type limit: int or None
    """

    table: str
    conditions: list[str] = field(default_factory=list)
    limit: int | None = None

    def where(self, condition: str) -> "QueryBuilder":
        """Add a WHERE condition.

        :param condition: A SQL condition expression (e.g. ``"age > 18"``).
        :type condition: str
        :returns: A new builder with the condition appended.
        :rtype: QueryBuilder
        """
        return replace(self, conditions=[*self.conditions, condition])

    def with_limit(self, n: int) -> "QueryBuilder":
        """Set the LIMIT clause.

        :param n: Maximum number of rows.
        :type n: int
        :returns: A new builder with the limit set.
        :rtype: QueryBuilder
        """
        return replace(self, limit=n)

    def build(self) -> str:
        """Render the final SQL string.

        :returns: A SQL SELECT statement.
        :rtype: str
        """
        sql = f"SELECT * FROM {self.table}"
        if self.conditions:
            sql += " WHERE " + " AND ".join(self.conditions)
        if self.limit:
            sql += f" LIMIT {self.limit}"
        return sql
```

#### Singleton

In Python, **modules are singletons**. Prefer module-level state to a
Singleton class. If you must enforce a single instance, use `__new__`:

```python
class _Registry:
    _instance: "_Registry | None" = None

    def __new__(cls) -> "_Registry":
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```

### 10.2 Structural Patterns

#### Decorator (Wrapper)

Python's first-class functions and `functools.wraps` make this pattern trivial:

```python
import functools
import time

def retry(max_attempts: int = 3, delay: float = 1.0):
    """Decorator that retries a function on failure with exponential backoff.

    :param max_attempts: Total number of attempts before giving up.
    :type max_attempts: int
    :param delay: Base delay in seconds (doubled each retry).
    :type delay: float
    :returns: A decorator that wraps the target function.
    :rtype: Callable
    """
    if max_attempts < 1:
        raise ValueError("max_attempts must be >= 1")

    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            last_exc: Exception | None = None
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as exc:
                    last_exc = exc
                    if attempt < max_attempts - 1:
                        time.sleep(delay * (2 ** attempt))
            assert last_exc is not None  # loop ran at least once
            raise last_exc
        return wrapper
    return decorator

@retry(max_attempts=5)
def fetch_data(url: str) -> bytes:
    """Fetch raw bytes from the given URL.

    :param url: The URL to retrieve.
    :type url: str
    :returns: The response body.
    :rtype: bytes
    """
    ...
```

#### Adapter

Use a wrapper class or, for simple cases, a function:

```python
class LegacyPrinter:
    """Third-party printer with a non-standard interface."""

    def print_old(self, text: str) -> None: ...

class PrinterAdapter:
    """Adapt :class:`LegacyPrinter` to the standard ``Printer`` interface.

    :param legacy: The legacy printer instance to wrap.
    :type legacy: LegacyPrinter
    """

    def __init__(self, legacy: LegacyPrinter):
        self._legacy = legacy

    def print(self, text: str) -> None:
        """Print text via the legacy printer.

        :param text: The text to print.
        :type text: str
        """
        self._legacy.print_old(text)
```

#### Facade

Expose a simplified interface to a complex subsystem:

```python
class EmailService:
    """Simplified interface for sending templated emails.

    :param smtp: The SMTP client used to deliver messages.
    :type smtp: SmtpClient
    :param template_engine: Engine used to render email templates.
    :type template_engine: TemplateEngine
    """

    def __init__(self, smtp: SmtpClient, template_engine: TemplateEngine):
        self._smtp = smtp
        self._templates = template_engine

    def send_welcome(self, user: User) -> None:
        """Send a welcome email to a newly registered user.

        :param user: The user to welcome.
        :type user: User
        """
        body = self._templates.render("welcome", user=user)
        self._smtp.send(to=user.email, subject="Welcome!", body=body)
```

### 10.3 Behavioural Patterns

#### Strategy

Use `Protocol` or `Callable` — no class hierarchy needed for simple cases:

```python
import gzip
from typing import Protocol

class Compressor(Protocol):
    """Interface for data compression strategies."""

    def compress(self, data: bytes) -> bytes:
        """Compress raw data.

        :param data: The uncompressed input.
        :type data: bytes
        :returns: The compressed output.
        :rtype: bytes
        """
        ...

class GzipCompressor:
    """Compress data using gzip."""

    def compress(self, data: bytes) -> bytes:
        """Compress data with gzip.

        :param data: The uncompressed input.
        :type data: bytes
        :returns: Gzip-compressed bytes.
        :rtype: bytes
        """
        return gzip.compress(data)

class Archiver:
    """Archive payloads using a pluggable compression strategy.

    :param compressor: The compression strategy to use.
    :type compressor: Compressor
    """

    def __init__(self, compressor: Compressor):
        self._compressor = compressor

    def archive(self, payload: bytes) -> bytes:
        """Compress and return the given payload.

        :param payload: Raw data to archive.
        :type payload: bytes
        :returns: Compressed data.
        :rtype: bytes
        """
        return self._compressor.compress(payload)
```

#### Observer / Event System

Use callbacks, signals, or a lightweight event emitter:

```python
from collections.abc import Callable


class EventEmitter:
    """A lightweight publish-subscribe event system."""

    def __init__(self) -> None:
        """Initialise an empty listener registry."""
        self._listeners: dict[str, list[Callable]] = {}

    def on(self, event: str, callback: Callable) -> None:
        """Register a callback for an event.

        :param event: The event name to subscribe to.
        :type event: str
        :param callback: The function to call when the event fires.
        :type callback: Callable
        """
        self._listeners.setdefault(event, []).append(callback)

    def emit(self, event: str, *args, **kwargs) -> None:
        """Fire an event, calling all registered listeners.

        :param event: The event name to emit.
        :type event: str
        :param args: Positional arguments forwarded to each listener.
        :param kwargs: Keyword arguments forwarded to each listener.
        """
        for callback in self._listeners.get(event, []):
            callback(*args, **kwargs)
```

#### Iterator

Implement `__iter__` and `__next__`, or — more idiomatically — use a
**generator** (PEP 255):

```python
from collections.abc import Iterator


def fibonacci() -> Iterator[int]:
    """Generate the Fibonacci sequence indefinitely.

    :returns: An infinite iterator of Fibonacci numbers.
    :rtype: Iterator[int]
    """
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b
```

#### Template Method

Use an abstract base class with concrete steps:

```python
from abc import ABC, abstractmethod

class DataPipeline(ABC):
    """Abstract ETL pipeline with fixed extract-transform-load steps."""

    def run(self, source: str) -> None:
        """Execute the full pipeline.

        :param source: Path or URI to the data source.
        :type source: str
        """
        raw = self.extract(source)
        clean = self.transform(raw)
        self.load(clean)

    @abstractmethod
    def extract(self, source: str) -> bytes:
        """Read raw data from the source.

        :param source: Path or URI to the data source.
        :type source: str
        :returns: Raw bytes from the source.
        :rtype: bytes
        """
        ...

    @abstractmethod
    def transform(self, raw: bytes) -> dict:
        """Transform raw bytes into a structured dictionary.

        :param raw: The raw data to transform.
        :type raw: bytes
        :returns: Cleaned, structured data.
        :rtype: dict
        """
        ...

    @abstractmethod
    def load(self, data: dict) -> None:
        """Persist the transformed data to the destination.

        :param data: The structured data to load.
        :type data: dict
        """
        ...
```

---

## 11. Testing

### 11.1 Principles

- Follow the **Arrange-Act-Assert** (AAA) pattern.
- Tests are **first-class code** — apply the same naming, formatting, and SRP
  standards.
- **Test behaviour, not implementation.** Tests should survive refactors that
  preserve behaviour.
- Aim for a **testing pyramid**: many unit tests, fewer integration tests,
  minimal end-to-end tests.

### 11.2 Naming

Test names should describe the **scenario** and the **expected outcome**:

```python
def test_withdraw_insufficient_funds_raises_error():
    ...

def test_parse_csv_with_empty_file_returns_empty_list():
    ...
```

### 11.3 Structure

```python
def test_apply_discount_reduces_total():
    # Arrange
    cart = ShoppingCart(items=[Item("Book", price=20.00)])
    discount = PercentageDiscount(10)

    # Act
    cart.apply_discount(discount)

    # Assert
    assert cart.total == 18.00
```

### 11.4 Fixtures and Factories

- Use **pytest fixtures** for reusable setup/teardown.
- Use **factory functions** or libraries like `factory_boy` to build test data
  rather than scattering raw constructors.

### 11.5 Mocking

- Prefer **dependency injection** over patching. If you must patch, patch where
  the name is **looked up**, not where it is defined.
- Only mock at **system boundaries** — external APIs, file systems, clocks.
  Don't mock your own domain objects.

---

## 12. Database Access & ACID

When interacting with SQL databases, every query and transaction must respect
the **ACID** properties — **Atomicity**, **Consistency**, **Isolation**, and
**Durability**. Violations cause data corruption, phantom reads, and silent
data loss — bugs that are extraordinarily hard to diagnose after the fact.

### 12.1 ACID at a Glance

| Property | Guarantee | Violation Example |
|---|---|---|
| **Atomicity** | A transaction either completes entirely or has no effect. | A transfer debits one account but crashes before crediting the other. |
| **Consistency** | A transaction moves the database from one valid state to another; all constraints hold. | An insert succeeds despite violating a foreign key constraint. |
| **Isolation** | Concurrent transactions do not interfere with each other. | Two threads read the same balance, both withdraw, and the final balance is wrong. |
| **Durability** | Once committed, data survives crashes and power loss. | A commit returns successfully but the data is lost after a restart. |

### 12.2 Always Use Explicit Transactions

Never rely on auto-commit for multi-statement operations. Wrap related writes
in a single transaction so they succeed or fail as a unit:

```python
from contextlib import contextmanager

@contextmanager
def transaction(conn):
    """Execute a block inside a database transaction.

    :param conn: An active database connection.
    :type conn: Connection
    :returns: A context manager yielding the connection.
    :rtype: Iterator[Connection]
    :raises Exception: Re-raises any exception after rolling back.
    """
    try:
        yield conn
        conn.commit()
    except Exception:
        conn.rollback()
        raise


def transfer(conn, from_id: int, to_id: int, amount: float) -> None:
    """Transfer funds between two accounts atomically.

    :param conn: An active database connection.
    :type conn: Connection
    :param from_id: The source account ID.
    :type from_id: int
    :param to_id: The destination account ID.
    :type to_id: int
    :param amount: The amount to transfer.
    :type amount: float
    :raises InsufficientFundsError: If the source account lacks funds.
    """
    with transaction(conn):
        cursor = conn.cursor()
        cursor.execute(
            "UPDATE accounts SET balance = balance - %s WHERE id = %s",
            (amount, from_id),
        )
        cursor.execute(
            "UPDATE accounts SET balance = balance + %s WHERE id = %s",
            (amount, to_id),
        )
```

### 12.3 Choose the Correct Isolation Level

Most databases default to READ COMMITTED, which prevents dirty reads but
allows non-repeatable reads and phantoms. Choose the level appropriate to the
operation:

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Use When |
|---|---|---|---|---|
| READ UNCOMMITTED | Possible | Possible | Possible | Almost never — analytics on stale data at most |
| READ COMMITTED | No | Possible | Possible | Default for most OLTP workloads |
| REPEATABLE READ | No | No | Possible | Reports or computations that must see a stable snapshot |
| SERIALIZABLE | No | No | No | Financial transactions, inventory, anything where correctness is paramount |

Set the isolation level explicitly when the default is insufficient:

```python
conn.set_session(isolation_level="SERIALIZABLE")
```

### 12.4 Use Parameterised Queries — Never String Interpolation

This protects both **consistency** (correct types) and **security** (SQL
injection):

```python
# Yes — parameterised
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))

# No — string interpolation (SQL injection risk, type errors)
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
```

### 12.5 SQL Injection Protection

SQL injection remains one of the most common and dangerous vulnerabilities.
Every database interaction **must** be protected against it.

- **Always use parameterised queries** with `?` or `%s` placeholders. The
  database driver handles escaping and type conversion correctly.
- **Never use f-strings, `.format()`, or `%` string formatting** to build SQL
  statements. Even when the value "looks safe", string interpolation bypasses
  the driver's escaping and invites injection.

```python
# Bad — every one of these is vulnerable to SQL injection
cursor.execute(f"SELECT * FROM users WHERE name = '{name}'")
cursor.execute("SELECT * FROM users WHERE name = '%s'" % name)
cursor.execute("SELECT * FROM users WHERE name = '{}'".format(name))

# Good — parameterised query
cursor.execute("SELECT * FROM users WHERE name = %s", (name,))
```

- **Use ORM query builders** (SQLAlchemy, Django ORM) which parameterise
  automatically. Even with an ORM, avoid constructing raw SQL via string
  formatting:

```python
# Good — SQLAlchemy uses parameterised queries internally
users = session.query(User).filter(User.name == name).all()

# Bad — raw SQL with string formatting, even inside an ORM session
session.execute(f"SELECT * FROM users WHERE name = '{name}'")
```

- **Validate and constrain input** before it reaches the database layer.
  Restrict string lengths, check numeric ranges, and reject unexpected
  characters at the service boundary — not in the SQL query.

### 12.6 Handle Connection Lifecycle Properly

- **Use connection pools** (`psycopg2.pool`, SQLAlchemy engine pool) — never
  open a connection per query.
- **Close connections and cursors** deterministically via context managers.
- **Retry on transient errors** (connection reset, serialisation failure) with
  backoff, but **never silently swallow** the error.

### 12.7 ORM Transactions

When using an ORM (SQLAlchemy, Django ORM), the same ACID principles apply.
Use the ORM's transaction API rather than manual SQL:

```python
from sqlalchemy.orm import Session

def transfer(session: Session, from_id: int, to_id: int, amount: float) -> None:
    """Transfer funds between two accounts atomically.

    :param session: An active SQLAlchemy session.
    :type session: Session
    :param from_id: The source account ID.
    :type from_id: int
    :param to_id: The destination account ID.
    :type to_id: int
    :param amount: The amount to transfer.
    :type amount: float
    """
    with session.begin():
        source = session.get(Account, from_id, with_for_update=True)
        target = session.get(Account, to_id, with_for_update=True)
        source.balance -= amount
        target.balance += amount
```

- Use `SELECT ... FOR UPDATE` (or the ORM equivalent) to prevent concurrent
  modifications when reading values that will be updated within the same
  transaction.
- Keep transactions **short** — hold locks for the minimum time necessary.

---

## 13. Concurrency & Async (PEP 3156)

### 13.1 Choosing a Model

| Task Type | Recommended Approach |
|---|---|
| I/O-bound (HTTP, DB, files) | `asyncio` or threading |
| CPU-bound (compute, parsing) | `multiprocessing` or `concurrent.futures.ProcessPoolExecutor` |
| Mixed | `asyncio` with `loop.run_in_executor` for CPU tasks |

### 13.2 Async/Await Conventions

```python
import asyncio

import aiohttp


async def fetch_one(session: aiohttp.ClientSession, url: str) -> bytes:
    """Fetch the body of a single URL.

    :param session: The shared HTTP session.
    :type session: aiohttp.ClientSession
    :param url: The URL to retrieve.
    :type url: str
    :returns: The response body.
    :rtype: bytes
    """
    async with session.get(url) as response:
        response.raise_for_status()
        return await response.read()


async def fetch_all(urls: list[str]) -> list[bytes]:
    """Fetch all URLs concurrently using structured concurrency.

    :param urls: The URLs to retrieve.
    :type urls: list[str]
    :returns: A list of response bodies, one per URL, in input order.
    :rtype: list[bytes]
    """
    async with aiohttp.ClientSession() as session:
        # Read bodies *inside* the session — response objects are not valid
        # after the session is closed.
        async with asyncio.TaskGroup() as tg:  # Python 3.11+
            tasks = [tg.create_task(fetch_one(session, url)) for url in urls]
        return [t.result() for t in tasks]
```

- Mark functions `async` only if they perform actual async operations.
- **Never call blocking I/O** inside an async function without
  `run_in_executor`.
- Use `asyncio.TaskGroup` (Python 3.11+) for structured concurrency.

### 13.3 Thread Safety

- Protect shared mutable state with `threading.Lock` or use thread-safe
  data structures (`queue.Queue`).
- Prefer **immutable data** passed between threads.

---

## 14. Performance & Pythonic Idioms

### 14.1 Comprehensions Over Loops

```python
# Yes
squares = [x ** 2 for x in range(10) if x % 2 == 0]

# No
squares = []
for x in range(10):
    if x % 2 == 0:
        squares.append(x ** 2)
```

But keep comprehensions **simple** — if it doesn't fit on two lines, use a
loop or extract a function.

### 14.2 Built-in Functions

Prefer `any()`, `all()`, `sum()`, `min()`, `max()`, `sorted()`,
`enumerate()`, `zip()` over manual loops.

```python
if any(user.is_admin for user in users):
    grant_access()

for index, item in enumerate(items, start=1):
    print(f"{index}. {item}")
```

### 14.3 Unpacking

```python
first, *rest = scores
x, y, z = point

for key, value in mapping.items():
    ...
```

### 14.4 F-Strings (PEP 498)

```python
# Yes
message = f"Hello, {name}! You have {count} new messages."

# No
message = "Hello, %s! You have %d new messages." % (name, count)
message = "Hello, {}! You have {} new messages.".format(name, count)
```

### 14.5 Walrus Operator (PEP 572)

Use assignment expressions to avoid redundant computation, but only when they
improve readability:

```python
if (match := pattern.search(text)) is not None:
    process(match.group())

while (chunk := file.read(8192)):
    hasher.update(chunk)
```

### 14.6 Structural Pattern Matching (PEP 634)

Python 3.10+ provides `match`/`case` for clean multi-branch dispatch:

```python
match command:
    case {"action": "move", "direction": direction}:
        move(direction)
    case {"action": "stop"}:
        stop()
    case _:
        raise UnknownCommandError(command)
```

### 14.7 Guard Clauses

Exit early rather than nesting deeply:

```python
# Yes — guard clauses
def process_order(order: Order) -> Receipt:
    """Build a receipt for the given order.

    :param order: The order to process.
    :type order: Order
    :returns: A receipt summarising the order.
    :rtype: Receipt
    :raises OrderCancelledError: If the order has been cancelled.
    """
    if order.is_cancelled:
        raise OrderCancelledError(order.id)
    if not order.items:
        return Receipt.empty()
    return _build_receipt(order)

# No — deep nesting
def process_order(order: Order) -> Receipt:
    if not order.is_cancelled:
        if order.items:
            return _build_receipt(order)
        else:
            return Receipt.empty()
    else:
        raise OrderCancelledError(order.id)
```

### 14.8 Copy Semantics — Shallow Copy, Deep Copy & Circular References

Python variables hold **references** to objects, not the objects themselves.
Assignment (`b = a`) creates a new reference to the same object — it does
**not** copy data. Understanding when to copy and how to avoid circular
references is essential for correct, leak-free code.

#### Reference vs. Shallow Copy vs. Deep Copy

```python
import copy

original = {"users": [{"name": "Alice"}, {"name": "Bob"}]}

# Reference — same object, mutations visible everywhere
ref = original
ref["users"].append({"name": "Charlie"})
assert len(original["users"]) == 3  # original is mutated!

# Shallow copy — new outer container, inner objects still shared
shallow = copy.copy(original)
shallow["users"].append({"name": "Dana"})
assert len(original["users"]) == 4  # inner list is shared!

# Deep copy — fully independent clone, no shared references
deep = copy.deepcopy(original)
deep["users"].append({"name": "Eve"})
assert len(original["users"]) == 4  # original unchanged
```

| Operation | Outer container | Nested objects | Use when |
|---|---|---|---|
| `b = a` | Shared | Shared | You want an alias, not a copy |
| `copy.copy(a)` | New | Shared | Flat structures, immutable elements |
| `copy.deepcopy(a)` | New | New (recursive) | Nested mutable structures that must be independent |

Shorthand for common shallow copies:

```python
new_list = old_list[:]           # or old_list.copy() or list(old_list)
new_dict = {**old_dict}          # or old_dict.copy()
new_set  = old_set.copy()
```

#### When to Copy

- **Defensive copies at boundaries.** When a function receives a mutable
  container it should not modify, copy it:

```python
def process_items(items: list[str]) -> list[str]:
    """Process items without modifying the caller's list.

    :param items: The items to process.
    :type items: list[str]
    :returns: The processed items.
    :rtype: list[str]
    """
    working = items[:]  # shallow copy — safe since str is immutable
    working.sort()
    return working
```

- **Returning internal state.** Never expose a private mutable attribute
  directly — return a copy or an immutable view:

```python
class Registry:
    """A registry of named services."""

    def __init__(self) -> None:
        self._services: dict[str, Service] = {}

    @property
    def services(self) -> dict[str, Service]:
        """Return a snapshot of registered services.

        :returns: A shallow copy of the internal service mapping.
        :rtype: dict[str, Service]
        """
        return self._services.copy()
```

- **Default mutable arguments.** Never use mutable defaults — they are shared
  across all calls:

```python
# Bad — shared list across all calls
def append_to(item, target=[]):
    target.append(item)
    return target

# Good — create a new list each call
def append_to(item, target: list | None = None) -> list:
    """Append an item to the target list, or a new list if none is given.

    :param item: The item to append.
    :param target: The list to append to.
    :type target: list or None
    :returns: The list with the item appended.
    :rtype: list
    """
    if target is None:
        target = []
    target.append(item)
    return target
```

#### Circular References

Circular references occur when objects reference each other, forming a cycle.
Python's garbage collector handles most cycles, but they can still cause
problems:

- **Memory stays allocated longer** — cyclic garbage is only collected
  periodically, not immediately via reference counting.
- **`copy.deepcopy` enters infinite recursion** if the cycle is not handled.
- **Serialization** (`pickle`, `json`) fails or produces corrupt output.
- **`__del__` finalizers** on objects in a cycle may never run (or run in
  unpredictable order).

**Preventing circular references:**

```python
import weakref

class Parent:
    """A parent node that owns its children."""

    def __init__(self, name: str) -> None:
        self.name = name
        self.children: list[Child] = []

    def add_child(self, child: "Child") -> None:
        """Add a child to this parent.

        :param child: The child to add.
        :type child: Child
        """
        self.children.append(child)

class Child:
    """A child node with a weak back-reference to its parent."""

    def __init__(self, name: str, parent: Parent) -> None:
        self._parent_ref = weakref.ref(parent)  # weak reference breaks the cycle
        self.name = name
        parent.add_child(self)

    @property
    def parent(self) -> Parent | None:
        """Return the parent, or None if it has been garbage-collected.

        :returns: The parent node.
        :rtype: Parent or None
        """
        return self._parent_ref()
```

**Rules of thumb:**

| Situation | Strategy |
|---|---|
| Parent → child ownership | Parent holds strong reference; child uses `weakref.ref` for back-reference |
| Caches and registries | Use `weakref.WeakValueDictionary` so entries are collected when no longer used |
| Observer/listener patterns | Store listeners as weak references so they don't pin objects in memory |
| Graph/tree structures | Choose one direction as "owning"; use weak references in the other |
| Debugging suspected cycles | Use `gc.get_referrers()` and `objgraph` to trace reference chains |

```python
import weakref

# Cache that doesn't prevent garbage collection
_cache: weakref.WeakValueDictionary[str, ExpensiveObject] = (
    weakref.WeakValueDictionary()
)
```

---

## 15. Defensive Programming & Input Validation

Trusting external input is the root cause of entire vulnerability classes —
injection attacks, buffer overflows, denial of service, and data corruption.
Validate **early**, at the boundary where data enters the system, so that
downstream code can operate on trusted, well-typed values.

### 15.1 Validate at System Boundaries

- **Validate all external input** — CLI arguments, API payloads, file content,
  environment variables, and inter-service messages. Internal function calls
  between trusted modules do not need redundant validation.
- Use **type hints and runtime validation** together. Type hints catch
  structural issues at development time; runtime validators (Pydantic,
  `isinstance()`, `typing.get_type_hints()`) catch bad data at execution time:

```python
from pydantic import BaseModel, Field

class CreateOrder(BaseModel):
    """Validated input for order creation."""

    product_id: int = Field(gt=0)
    quantity: int = Field(ge=1, le=1000)
    customer_email: str = Field(max_length=255)
```

### 15.2 Common Validation Patterns

- **Validate string lengths** to prevent excessive memory usage and storage
  overflow:

```python
if len(name) > 255:
    raise ValueError(f"Name exceeds maximum length: {len(name)}")
```

- **Validate numeric ranges** to reject nonsensical or dangerous values:

```python
if not (1 <= port <= 65535):
    raise ValueError(f"Invalid port number: {port}")
```

- **Validate with dataclass `__post_init__`** or Pydantic validators for
  domain objects that enforce their own invariants:

```python
from dataclasses import dataclass

@dataclass
class Port:
    """A validated TCP/UDP port number."""

    number: int

    def __post_init__(self) -> None:
        if not (1 <= self.number <= 65535):
            raise ValueError(f"Invalid port: {self.number}")
```

- **Use `enum.Enum`** for restricted value sets — never accept arbitrary strings
  when the domain has a fixed set of valid options:

```python
from enum import Enum

class Status(Enum):
    ACTIVE = "active"
    INACTIVE = "inactive"
    SUSPENDED = "suspended"
```

### 15.3 Defensive Access Patterns

- **Check collection bounds** before indexing. Use `.get()` with defaults for
  dicts rather than relying on `KeyError`:

```python
value = config.get("timeout", 30)  # safe default
first = items[0] if items else None  # bounds check
```

### 15.4 Input Sanitisation

- **Sanitize input used in file paths** to prevent directory traversal attacks.
  Use `os.path.normpath` and `pathlib` to resolve relative components:

```python
from pathlib import Path

safe_path = Path(base_dir) / Path(user_input).name  # strip directory components
```

- **Sanitize input used in shell commands** with `shlex.quote`. Better yet,
  avoid shell commands entirely and use `subprocess` with a list of arguments:

```python
import subprocess

# Good — no shell interpretation
subprocess.run(["grep", pattern, filename], check=True)

# Bad — user input in a shell command
subprocess.run(f"grep {pattern} {filename}", shell=True)
```

- **Never pass user input** to `eval()`, `exec()`, or `os.system()`. These
  functions execute arbitrary code and are inherently unsafe with untrusted
  data.

### 15.5 Assertions vs. Validation

- Use `assert` **only for internal invariants** — conditions that should be
  impossible if the code is correct. Assertions can be disabled with
  `python -O`, so they must never guard against invalid user input:

```python
# Good — internal invariant
assert len(self._items) == self._count, "item list out of sync with count"

# Bad — input validation via assert (bypassed with -O)
assert len(username) < 256, "username too long"
```

---

## 16. Project Structure & Packaging (PEP 517 / 621)

### 16.1 Recommended Layout

```
my_project/
    pyproject.toml          # PEP 621 — single source of project metadata
    README.md
    LICENSE
    src/
        my_package/
            __init__.py
            core.py
            models.py
            services/
                __init__.py
                auth.py
    tests/
        conftest.py
        test_core.py
        test_models.py
        services/
            test_auth.py
```

- Use the **src layout** (recommended by the Python Packaging Authority) — it
  prevents accidental imports from the working directory. PEP 517 itself
  covers build-system interfaces, not project layout.
- Mirror the source tree structure in tests.

### 16.2 pyproject.toml (PEP 621)

`pyproject.toml` is the standard for project metadata and build configuration:

```toml
[project]
name = "my-package"
version = "1.0.0"
requires-python = ">=3.11"
dependencies = [
    "httpx>=0.27",
    "pydantic>=2.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "mypy>=1.10",
    "ruff>=0.4",
]
```

### 16.3 Dependency Management

- **Pin exact versions** in lock files (`uv.lock`, `poetry.lock`); use
  **compatible ranges** in `pyproject.toml`.
- Separate production and development dependencies.
- Audit dependencies for security vulnerabilities regularly.

---

## 17. Tooling

### 17.1 Recommended Tool Chain

| Purpose | Tool | Notes |
|---|---|---|
| Formatter | **Ruff** (or Black) | Deterministic formatting, end debates |
| Linter | **Ruff** (or Flake8 + plugins) | Covers PEP 8 + common bugs |
| Type checker | **mypy** or **pyright** | Run in strict mode in CI |
| Import sorter | **Ruff** (isort rules) | Enforce consistent import order |
| Test runner | **pytest** | With `pytest-cov` for coverage |
| Pre-commit | **pre-commit** | Run formatters and linters on every commit |
| Package manager | **uv** (or pip + pip-tools) | Fast, reliable dependency resolution |

### 17.2 Configuration

Centralise all tool configuration in `pyproject.toml`:

```toml
[tool.ruff]
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "UP", "B", "SIM", "RUF"]

[tool.mypy]
strict = true
warn_return_any = true
disallow_untyped_defs = true

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-ra --strict-markers"
```

### 17.3 Pre-commit Configuration

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.4.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.10.0
    hooks:
      - id: mypy
        additional_dependencies: [pydantic]
```

---

## 18. Build Tools

### 18.1 uv (Recommended Modern Tool)

**uv** is the recommended package manager (see §17.1). For Poetry users, the
following section shows the equivalent Poetry workflow.

```bash
uv init             # Create pyproject.toml + .python-version
uv add requests     # Add dependency (updates pyproject.toml + uv.lock)
uv sync             # Install from lock file
uv build            # Build wheel and sdist
uv publish          # Publish to PyPI
```

### 18.2 Poetry (Alternative)

Poetry manages dependencies, virtual environments, and packaging:

```bash
poetry init  # Create pyproject.toml
poetry add requests django  # Add dependencies
poetry lock  # Lock dependency versions
poetry install  # Install from lock file
poetry build  # Build wheel and sdist
poetry publish  # Publish to PyPI
```

Configuration in `pyproject.toml`:
```toml
[tool.poetry]
name = "my-app"
version = "0.1.0"
description = "My application"
authors = ["Author <author@example.com>"]

[tool.poetry.dependencies]
python = "^3.10"
requests = "^2.31.0"
django = "^4.2"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

### 18.3 pip + setuptools (Traditional)

`setup.py` (deprecated, use `setup.cfg` or `pyproject.toml` instead):
```python
from setuptools import setup, find_packages

setup(
    name='my-app',
    version='0.1.0',
    packages=find_packages(),
    install_requires=['requests>=2.31.0', 'django>=4.2'],
)
```

Build and publish:
```bash
pip install build twine
python -m build  # Creates dist/ with wheel and sdist
twine upload dist/*  # Upload to PyPI
```

### 18.4 Hatch (Modern Alternative)

Hatch provides project scaffolding, dependency management, and testing:

```bash
hatch new my-app  # Create new project
hatch env create  # Create environments
hatch test  # Run tests in isolation
hatch build  # Build distributions
```

### 18.5 Build Configuration

Standard `pyproject.toml` (PEP 517/518):
```toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "my-app"
version = "0.1.0"
description = "My application"
dependencies = ["requests>=2.31.0"]
```

### 18.6 Build Scripts with Make or Invoke

Makefile:
```makefile
.PHONY: test build publish

test:
	pytest -v

build:
	python -m build

publish:
	twine upload dist/*
```

Or use `invoke` for Python-based task automation:
```bash
pip install invoke
# Define tasks in tasks.py
invoke test
invoke build
```

### 18.7 Packaging & Distribution

Standard distribution formats:
- **wheel** (`.whl`): Binary package, fast installation
- **sdist** (`.tar.gz` or `.zip`): Source distribution, requires build

Publish to PyPI:
```bash
twine upload -r testpypi dist/*  # Test first
twine upload dist/*  # Production PyPI
```

---

## 19. SBOM Creation

### 19.1 What is an SBOM?

A Software Bill of Materials (SBOM) documents all dependencies, transitive packages, and frameworks. Essential for security audits, license compliance, and supply chain visibility.

### 19.2 pip-audit and SBOM Generation

**Using `pip-audit`** (scans for known vulnerabilities):
```bash
pip-audit --desc  # Shows descriptions of vulnerabilities
pip-audit --fix   # Auto-fix vulnerable packages
```

**CycloneDX for Python**:
```bash
pip install cyclonedx-bom
cyclonedx-py -o sbom.xml
cyclonedx-py -o sbom.json -of json
```

### 19.3 Poetry SBOM Generation

If using Poetry for dependency management:
```bash
poetry export --format requirements.txt > requirements.txt
# Then use cyclonedx on the output
```

### 19.4 Vulnerability Scanning

**Using `safety`** (checks against known vulnerabilities):
```bash
safety check
safety check --json > safety-report.json
```

Or use GitHub Dependabot / GitLab Dependency Scanning for continuous monitoring.

### 19.5 License Compliance

**Using `pip-licenses`**:
```bash
pip install pip-licenses
pip-licenses --format=json > licenses.json
pip-licenses --fail-on=GPL  # Fail if GPL packages detected
```

### 19.6 Integration into CI/CD

- Generate SBOM in CI on every release
- Run `pip-audit` to detect known vulnerabilities
- Check licenses with `pip-licenses`
- Store SBOM and audit results (JSON/XML) as release artifacts
- Use GitHub / GitLab dependency scanning dashboards for visibility

---

## 20. References

### Standards & PEPs

| PEP | Title |
|---|---|
| [PEP 8](https://peps.python.org/pep-0008/) | Style Guide for Python Code |
| [PEP 20](https://peps.python.org/pep-0020/) | The Zen of Python |
| [PEP 257](https://peps.python.org/pep-0257/) | Docstring Conventions |
| [PEP 328](https://peps.python.org/pep-0328/) | Imports: Multi-Line and Absolute/Relative |
| [PEP 484](https://peps.python.org/pep-0484/) | Type Hints |
| [PEP 498](https://peps.python.org/pep-0498/) | Literal String Interpolation (f-strings) |
| [PEP 517](https://peps.python.org/pep-0517/) | Build System Interface |
| [PEP 526](https://peps.python.org/pep-0526/) | Variable Annotations |
| [PEP 544](https://peps.python.org/pep-0544/) | Protocols: Structural Subtyping |
| [PEP 557](https://peps.python.org/pep-0557/) | Data Classes |
| [PEP 572](https://peps.python.org/pep-0572/) | Assignment Expressions (Walrus Operator) |
| [PEP 604](https://peps.python.org/pep-0604/) | Union Type Syntax (`X \| Y`) |
| [PEP 612](https://peps.python.org/pep-0612/) | Parameter Specification Variables |
| [PEP 621](https://peps.python.org/pep-0621/) | Project Metadata in `pyproject.toml` |
| [PEP 634](https://peps.python.org/pep-0634/) | Structural Pattern Matching |
| [PEP 695](https://peps.python.org/pep-0695/) | Type Parameter Syntax |
| [PEP 3134](https://peps.python.org/pep-3134/) | Exception Chaining |

### Books

| Book | Authors | Key Takeaways for This Guide |
|---|---|---|
| *Design Patterns: Elements of Reusable Object-Oriented Software* | Gamma, Helm, Johnson, Vlissides (1994) | Favour composition over inheritance; program to interfaces; encapsulate what varies. |
| *Clean Code: A Handbook of Agile Software Craftsmanship* | Robert C. Martin (2008) | Small functions, meaningful names, minimal comments, SRP, error handling discipline. |
| *The Pragmatic Programmer* | Hunt & Thomas (1999, 2019) | DRY, orthogonality, tracer bullets, pragmatic testing. |
| *Refactoring: Improving the Design of Existing Code* | Martin Fowler (1999, 2018) | Systematic code improvement; extract method, replace conditional with polymorphism. |
| *Effective Python* | Brett Slatkin (2015, 2019) | 90 Pythonic best practices covering generators, decorators, concurrency, and more. |
| *Fluent Python* | Luciano Ramalho (2015, 2022) | Deep dive into Python's data model, iterators, protocols, and metaprogramming. |
