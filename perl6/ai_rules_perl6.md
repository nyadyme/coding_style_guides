# Raku (Perl 6) — AI Coding Rules

Apply these rules when generating or reviewing Raku code.

## Boilerplate

- `use v6.d;` to pin language version.
- File extensions: `.rakumod` for modules, `.raku` for scripts, `.rakutest` for tests.
- Module metadata in `META6.json`.

## Formatting

- 4 spaces indentation. Never tabs.
- 100 characters recommended line limit.
- Opening brace on same line.
- Two blank lines between top-level subroutine/method definitions.
- One blank line between methods within a class.
- One blank line after `use` statements before first code.
- No blank line at start or end of a block.

## Naming

- `PascalCase::Separated` for modules/classes.
- `PascalCase` for roles and grammars.
- `kebab-case` for subroutines, methods, variables.
- `$kebab-case`, `@kebab-case`, `%kebab-case` for variables.
- `is-`, `has-`, `can-` prefix for booleans.
- `!` twigil for private attributes: `$!cache`.
- `.` twigil for public accessors: `$.name`.
- `kebab-case` for tokens and rules in grammars.

## Subroutines & Methods

- Small (under 20 lines), one thing, one level of abstraction.
- Type constraints in signatures: `sub connect(Str $host, Int $port, Int :$timeout = 30)`.
- Named parameters (`:$param`) for optional arguments.
- `where` clauses for value constraints.
- Multi dispatch for polymorphic behaviour: `multi sub process(Int $n)`, `multi sub process(Str $s)`.
- Explicit return types: `sub find-user(Int $id --> User)`.
- Return `Nil` for absence. `fail` for soft failures. `die` for hard errors.

## Object-Oriented Raku

- Built-in OO with `class`, `has`, `method`.
- `$.attr` for public accessor, `$!attr` for private.
- `is required` for mandatory attributes.
- Roles (`does RoleName`) over inheritance (`is ParentClass`).
- Roles detect conflicts at composition time.
- `BUILD` or `TWEAK` submethods for custom construction.
- Runtime mixins with `but` keyword.

## Typed Data Passing

- Pass typed objects, not raw hashes or arrays, between methods and modules.
- Use type constraints in signatures to enforce data contracts.
- At system boundaries (API responses, config files, CLI args), parse into typed objects immediately.
- Return typed objects with explicit return type annotations (`-->`).

## SOLID

- **S**: One class per concern.
- **O**: Extend via roles, multi dispatch, and augment.
- **L**: Subtypes must honour the base type's interface.
- **I**: Keep roles small and focused.
- **D**: Inject dependencies via constructor attributes.

## Documentation

- `#|` declarator block before declaration, `#=` after declaration.
- Documentation accessible at runtime via `.WHY`.
- Pod6 for longer documentation: `=begin pod` / `=end pod`.
- `=head1`, `=head2` for section structure.

## Error Handling

- Typed exceptions: `class X::App::NotFound is Exception { ... }`.
- `die` for unrecoverable errors. `fail` for lazy/soft failures.
- `CATCH` phaser for structured error handling within `try` blocks.
- Catch by type: `when X::App::NotFound { ... }`.
- `LEAVE` phaser for resource cleanup.
- `.rethrow` to re-throw caught exceptions.

## Imports

- Order: pragmas (`use v6.d;`), core modules, ecosystem modules (Zef), project modules.
- `is export` trait selectively — don't export everything.
- One module per concern: one HTTP client, one JSON library, one database approach.

## Concurrency

- `start` blocks and `await` for async work.
- `react`/`whenever` for event-driven code.
- Channels for message passing between concurrent tasks.
- Supplies for reactive streams.
- `hyper` (ordered) and `race` (unordered) for data parallelism.
- Never share mutable state. Use channels or `Lock::Async`.

## Database / ACID

- DBIish for database access.
- Explicit transactions: `BEGIN`, `COMMIT`, `ROLLBACK`.
- Parameterised queries with placeholders. Never interpolate SQL.
- Never interpolate or concatenate variables into SQL strings. Always use `?` placeholders via DBIish.
- Validate and constrain input before it reaches the database layer.

## Testing

- Built-in `Test` module or `Test::Async`.
- `plan N` or `done-testing`.
- `subtest 'description' => { ... }` for grouped tests.
- `ok`, `is`, `isnt`, `like`, `dies-ok`, `throws-like`, `isa-ok` assertions.
- Role-based dependency injection for test doubles.

## Type System

- Gradual typing: add types where they improve clarity.
- `subset` for constrained types: `subset PositiveInt of Int where * > 0;`.
- Junctions for multi-value matching: `any(1, 2, 3)`, `none(4, 5)`.

## Grammars

- `grammar Name { token TOP { ... } }` for parsing.
- `token` (non-backtracking), `rule` (whitespace-significant), `regex` (backtracking).
- Actions class for semantic processing of parsed results.

## Performance & Idioms

- `given`/`when` for smart matching (switch).
- `gather`/`take` for lazy list generation.
- `...` (sequence operator) for series.
- `»` (hyper operator) for element-wise operations.
- Feed operators (`==>`) for pipeline-style chaining.
- Reduction meta-operator: `[+] @numbers`.
- Function composition with `o` operator.
- Guard clauses for early returns.

## Defensive Programming & Input Validation

- Validate all external input at method/subroutine boundaries.
- Use type constraints in signatures: `sub process(Str $name where *.chars <= 255)`.
- Use `subset` types for reusable validation: `subset Port of Int where 1..65535;`.
- Use `where` clauses for value constraints.
- Use multi dispatch for handling different input types safely.
- Check collection bounds before indexing.
- Sanitize input used in file paths, system commands, and regex.
- Never use `EVAL` with unsanitized user input.
- Use the gradual type system: add types at boundaries where external data enters.

## Project Structure

- `lib/` for modules, `t/` for tests, `bin/` for executables, `resources/` for data files.
- `META6.json` declares name, version, auth, dependencies, provides.
- Zef for package management.

## Tooling

- Zef (package manager), prove6 (test runner).
- Pod6 / `raku --doc` (documentation), Telemetry (profiling).

## Build Tools

- Zef is the native package manager and build tool.
- META6.json declares module metadata and dependencies.
- `zef build`, `zef test`, `zef upload` for building/testing/publishing.
- Use `prove6` as test harness for .rakutest files.
- Plain `Makefile` or shell scripts for task automation — Raku has no standard task runner (no Rakefile/Rakufile equivalent).
- Docker for containerized Raku applications.

## SBOM Creation

- Declare dependencies in `META6.json` with exact `:ver<>` and `:auth<>`.
- Zef has no lock file — pin versions in `META6.json` or vendor dependencies for reproducibility.
- Monitor Raku ecosystem for security advisories.
- Generate SBOM on release.
- Verify licenses of all dependencies.
