# Python — AI Coding Rules

Apply these rules when generating or reviewing Python code.

## Python Version

- **Python 3 only.** Target Python 3.9+ (3.11+ recommended).
- Python 2 is end-of-life (sunset 2020-01-01). Never generate Python 2 code.
- Never use Python 2-only constructs: `print` statement, `xrange`, `iteritems`, `basestring`, `unicode`, `raw_input`, `__metaclass__`, `cPickle`, `urllib2`, etc.
- No `from __future__ import` is needed in Python 3.
- Always use `#!/usr/bin/env python3` (never `python` or `python2`).

## Formatting

- 4 spaces indentation. Never tabs.
- 79 characters line limit (72 for docstrings/comments). Teams may use 88 (Black).
- Two blank lines before/after top-level functions and classes.
- One blank line between methods inside a class.
- No blank line after `def`, `if`, `for`, `while`, `with`.
- Trailing commas in multi-line structures.
- Always `#!/usr/bin/env python3` shebang for executable scripts.
- Always `if __name__ == "__main__": main()` guard. Extract logic into `main()`.

## Naming

- `snake_case`: functions, methods, variables, modules.
- `PascalCase`: classes, exceptions (suffix `Error`), type variables.
- `UPPER_SNAKE_CASE`: constants.
- Single `_` prefix: private. Double `__` prefix: name-mangled.
- Intention-revealing, pronounceable, searchable. No Hungarian notation.

## Functions

- Small (under 20 lines), one thing, one level of abstraction.
- Max 3 positional parameters. Beyond that, use a dataclass or typed dict.
- No boolean flag arguments — split into two functions.
- Use keyword-only arguments (`*`) for clarity.
- Command-Query Separation: commands return `None`, queries have no side effects.
- Avoid lambdas. Use named functions or `operator.attrgetter`/`itemgetter`/`methodcaller`.
- Prefer comprehensions over `map()`/`filter()` with lambdas.

## Types

- Annotate all public function signatures.
- Built-in generics: `list[int]`, `dict[str, Any]` (3.9+).
- Union shorthand: `int | None` (3.10+).
- Import from `collections.abc`: `Callable`, `Iterable`, `Sequence`, `Mapping`.
- Use `Protocol` for structural subtyping.
- Use `@dataclass(frozen=True)` for immutable data containers.
- Pass typed objects (dataclasses, Pydantic models, NamedTuples) between functions — not raw `dict` or strings.
- At system boundaries (API responses, config files, CLI args), parse into typed objects immediately.
- Return typed objects, not tuples of loosely related values.

## Docstrings (reStructuredText format)

- Triple double-quotes. First line: imperative summary.
- Use `:param name:`, `:type name:`, `:returns:`, `:rtype:`, `:raises ExcType:`.
- All public modules, classes, functions, and methods require docstrings.

## Error Handling

- Raise exceptions, not return codes. Catch specific exceptions.
- Chain exceptions: `raise NewError(...) from exc`.
- Always use context managers (`with`) for resources: files, connections, locks, temp files.
- Use `contextlib.contextmanager` for simple acquire/release patterns.
- Use `contextlib.suppress()`, `closing()`, `ExitStack()` as appropriate.
- Use `async with` and `asynccontextmanager` for async resources.
- Use `logging`, never `print()`, for operational output.

## Imports

- Group: stdlib, third-party, local — separated by blank lines, alphabetical within.
- Absolute imports preferred. No `import *`.
- One module per concern: don't mix `time`/`datetime`, `os.path`/`pathlib`, `json`/`simplejson`.

## Classes

- Small in responsibilities, high cohesion.
- Composition over inheritance. Use `Protocol`/`ABC` for interfaces.
- SOLID principles. Inject dependencies through constructors.

## Copy Semantics

- Assignment creates references, not copies.
- `copy.copy()` for shallow, `copy.deepcopy()` for nested mutable structures.
- Use `weakref.ref` / `WeakValueDictionary` to break circular references.
- Never use mutable default arguments: `def f(items=None)` not `def f(items=[])`.
- Return copies of internal mutable state, not references.

## Testing

- Arrange-Act-Assert. `pytest` with fixtures.
- Name: `test_<scenario>_<expected_outcome>`.
- Mock only at system boundaries. Prefer dependency injection.

## Database

- Explicit transactions. Never auto-commit for multi-statement ops.
- Parameterised queries only — never string interpolation.
- Never use f-strings, `.format()`, or `%` to build SQL. Always use `%s`/`?` placeholders.
- Use ORM query builders (SQLAlchemy, Django ORM) which parameterise automatically.
- Validate and constrain input before it reaches the database layer.
- Use connection pools. Close via context managers.
- Set isolation level explicitly when READ COMMITTED is insufficient.

## Concurrency

- `asyncio` for I/O-bound. `multiprocessing` for CPU-bound.
- Never call blocking I/O inside `async` without `run_in_executor`.
- Protect shared state with `threading.Lock`. Prefer immutable data.

## Patterns

- Factory: `@classmethod` or standalone function.
- Builder: `@dataclass` with `replace()`.
- Singleton: module-level state.
- Decorator: `functools.wraps`. Strategy: `Protocol` or `Callable`.
- Iterator: generators with `yield`.

## Defensive Programming

- Validate all external input at system boundaries (CLI args, API payloads, file content, env vars).
- Use Pydantic or dataclass `__post_init__` for runtime validation. Validate string lengths, numeric ranges.
- Use `enum.Enum` for restricted value sets. Check collection bounds before indexing.
- Sanitize input in file paths (`pathlib`), URLs, and shell commands (`shlex.quote`).
- Never pass user input to `eval()`, `exec()`, or `os.system()`.
- Use `assert` only for internal invariants, never for input validation (can be disabled with `-O`).

## Tooling

- Formatter: Ruff (or Black). Linter: Ruff. Types: mypy/pyright strict.
- Tests: pytest. Imports: isort (via Ruff). Pre-commit hooks.
- All config in `pyproject.toml`.

## Build Tools

- Use Poetry for modern dependency management and packaging.
- Use `pyproject.toml` (PEP 517/518) for build configuration.
- `setuptools` + `wheel` for traditional builds.
- Use `python -m build` to create wheel and sdist distributions.
- `twine` for secure PyPI uploads.
- Use `invoke` or `make` for task automation (test, build, publish).

## SBOM Creation

- Use `cyclonedx-bom` for CycloneDX SBOM generation (JSON/XML).
- Run `pip-audit` to scan for known vulnerabilities.
- Use `pip-licenses` for license compliance checks.
- Use `safety check` as alternative vulnerability scanner.
- Generate SBOM on every release; store with artifacts.
- Integrate into CI/CD: scan on commit, enforce on release.
