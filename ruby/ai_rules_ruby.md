# Ruby — AI Coding Rules

Apply these rules when generating or reviewing Ruby code.

## Formatting

- 2 spaces indentation. Never tabs.
- 120 characters line limit (RuboCop `Layout/LineLength`).
- One blank line between method definitions.
- No blank line after class/module/method opening or before `end`.
- Two blank lines between class/module definitions in the same file.
- No blank line after `begin` or before `rescue`/`ensure`/`end`.
- Trailing commas in multi-line arrays, hashes, and argument lists.
- `# frozen_string_literal: true` at top of every file.
- Single quotes for plain strings, double quotes for interpolation/escapes.
- Heredocs (`<<~MSG`) for multi-line strings.
- RuboCop is the authority.

## Naming

- `PascalCase`: classes, modules.
- `snake_case`: methods, variables, file names.
- `UPPER_SNAKE_CASE`: constants (freeze mutable constants).
- `@snake_case`: instance variables. Avoid `@@` class variables.
- Predicate methods end with `?`. Dangerous methods end with `!`.
- Acronyms follow PascalCase: `HttpClient`, not `HTTPClient`.

## Methods

- Small — aim for 5-10 lines (Sandi Metz's rules).
- Max 2 positional params. Beyond that, use keyword arguments.
- No boolean flag arguments — split into two methods.
- Implicit return for last expression. Explicit `return` only for guard clauses.
- Default to `private`. Group `private` at bottom of class.
- Prefer named methods over lambdas for non-trivial logic.
- Use `method(:name)` and `&:method_name` for passing callables.

## Classes & Composition

- Single Responsibility. Small in responsibilities, high cohesion.
- Composition over inheritance. Dependency injection.
- Modules for namespacing and stateless shared behaviour (mixins).
- Keep mixins stateless — no instance variables.
- Duck typing over `is_a?` checks.
- `Data.define` (3.2+) for immutable value objects. `Struct` for mutable.
- Pass typed objects (`Struct`, `Data.define`, custom classes) between methods — not raw Hashes.
- Define data objects for all data crossing class/module boundaries.
- At system boundaries (API responses, config files, CLI args), parse into typed objects immediately.

## Documentation (YARD)

- `# summary` as first line (imperative mood).
- `@param name [Type]`, `@return [Type]`, `@raise [ErrorType]`, `@example`.
- Document all public classes and methods.

## Error Handling

- Raise exceptions, not return codes. Rescue specific exceptions.
- Never `rescue Exception` — only `rescue StandardError` (or subclasses).
- Domain exceptions inherit from `StandardError` via project base class.
- Use `ensure` for cleanup. Prefer block-form APIs for automatic cleanup.
- Custom block-based APIs (`yield`/`ensure`) for resources needing cleanup.
- Use a logger, never `puts`/`p`, for operational output.

## Requires & Gems

- Group: stdlib, third-party gems, `require_relative` — separated by blank lines.
- One gem per concern: one HTTP client, one test framework, one template engine.
- Pin versions with `~>` in Gemfile. Commit `Gemfile.lock`. Run `bundle audit`.

## Blocks, Procs & Lambdas

- `{ }` for single-line blocks, `do...end` for multi-line.
- Prefer lambdas over `Proc.new` (strict arity, local return).
- `&:method_name` shorthand for simple method calls.

## Copy Semantics

- Assignment creates references. `.dup` for shallow copy. `.clone` preserves frozen state.
- `Marshal.load(Marshal.dump(obj))` for deep copy (slow, no procs/IO).
- `.freeze` for immutability. `deep_dup` (ActiveSupport) for nested structures.

## Testing

- RSpec or Minitest — one per project.
- `describe`/`context`/`it` structure. Arrange-Act-Assert.
- FactoryBot for test data. `instance_double` for type-checked mocks.
- Mock only at system boundaries.

## Database

- `ActiveRecord::Base.transaction` for atomic operations.
- Bang methods (`update!`, `save!`) inside transactions for rollback on failure.
- Parameterised queries: `where(email: email)` or `where("email = ?", email)`.
- `Account.lock.find(id)` for `SELECT ... FOR UPDATE`.
- Use framework connection pool. Never hold connections across request boundaries.
- Never use string interpolation in SQL (`"WHERE name = '#{name}'"` is forbidden).
- Use ActiveRecord hash conditions or `?` placeholders for all queries.
- Validate and constrain input before it reaches the database layer.

## Concurrency

- GVL limits CPU parallelism. Threads for I/O-bound work.
- `Mutex` for shared mutable state. Prefer immutable/frozen objects.
- Ractors (3+) for true parallelism. Sidekiq/GoodJob for background jobs.

## Patterns

- Factory: class methods. Builder: fluent interface.
- Singleton: `include Singleton`. Decorator: `SimpleDelegator`.
- Strategy: duck-typed objects. Observer: `Observable` or custom emitter.

## Defensive Programming

- Validate all external input at controller/service boundaries.
- `raise ArgumentError` for invalid input at method boundaries.
- Use `Kernel#Integer()`, `Kernel#Float()` for strict type conversion — not `to_i`/`to_f`.
- Validate string lengths and numeric ranges before processing or storing.
- Use `.fetch` with defaults for hash access — it raises `KeyError` on missing keys.
- Sanitize input in file paths (`File.expand_path`, `Pathname`) and shell commands (`Shellwords.escape`).
- Never use `eval`, `send`, or `instance_eval` with user input.
- Use `frozen_string_literal: true` to prevent accidental string mutation.
- Rails Strong Parameters for web input filtering.

## Tooling

- RuboCop (lint/format), RSpec/Minitest (test), SimpleCov (coverage).
- Brakeman (Rails security), bundler-audit (dependency vulns).
- Sorbet or RBS+Steep for gradual typing. YARD for docs.

## Build Tools

- Use Rake for task automation (build, test, release).
- Bundler locks dependencies in `Gemfile.lock` for reproducibility.
- Use `bundle exec` to run commands in Bundler environment.
- `.gemspec` defines gem metadata and dependencies.
- Rails 7+ defaults to Propshaft + import maps (Sprockets is legacy). Use jsbundling-rails (esbuild/rollup/webpack) or Shakapacker for modern JS bundling.
- Docker for containerized Ruby/Rails applications.

## SBOM Creation

- Lock `Gemfile.lock` in VCS for reproducible builds.
- Use `cyclonedx-ruby` for CycloneDX SBOM generation.
- Run `bundler-audit check --update` to scan for vulnerabilities.
- Use `license_finder` to verify license compliance.
- Gate CI/CD on `bundler-audit` and license checks.
- Store SBOM and audit reports with release artifacts.
