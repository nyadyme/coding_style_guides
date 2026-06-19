# Perl 5 — AI Coding Rules

Apply these rules when generating or reviewing Perl 5 code.

## Boilerplate (every file)

- `use strict; use warnings;` in every file. No exceptions.
- Or `use v5.36;` (implies strict, warnings, say).
- Executable scripts: `#!/usr/bin/env perl` shebang.
- Every module ends with `1;`.

## Formatting

- 4 spaces indentation. Never tabs.
- 80 characters line limit.
- K&R braces (opening brace on same line).
- Two blank lines between subroutine definitions.
- One blank line between logical sections within a subroutine.
- One blank line after `use` block before first code.
- No blank line at start or end of a block.
- Use Perl::Tidy for enforcement.

## Naming

- `PascalCase::Separated` for packages/modules.
- `snake_case` for subroutines, variables (`$scalar`, `@array`, `%hash`).
- `UPPER_SNAKE_CASE` for constants (`use constant MAX_RETRIES => 3;`).
- `_leading_underscore` for private subs/methods.
- `is_`, `has_`, `can_` prefix for booleans.
- `PascalCase.pm` for module files.
- No Hungarian notation — sigils already indicate type.

## Subroutines

- Small (under 20 lines), one thing, one level of abstraction.
- Unpack `@_` explicitly at the top: `my ($host, $port, %opts) = @_;`.
- Hash arguments for optional/named parameters.
- Validate arguments early.
- Perl 5.36+: use subroutine signatures `sub connect ($host, $port, %opts)`.
- Return explicitly: `return $result;`.
- Return `undef` for absence, `die` for errors.
- Avoid prototypes except for emulating built-in syntax.

## Object-Oriented Perl

- Use Moo (lightweight) or Moose (full-featured). No manual `bless`-based OO.
- `has name => (is => 'ro', isa => Str, required => 1);` for attributes.
- Roles (Moo::Role / Moose::Role) over deep inheritance.
- Roles detect conflicts at composition time.
- Use `Types::Standard` for type constraints.

## Typed Data Passing

- Pass Moo/Moose objects, not raw hashrefs or arrayrefs, between subroutines and modules.
- Define clear data classes for data crossing module boundaries.
- At system boundaries (API responses, config files, CLI args), parse into typed objects immediately.
- Use `Types::Standard` constraints to enforce data shapes.

## SOLID

- **S**: One module per concern.
- **O**: Extend via roles and composition.
- **L**: Subtypes must honour the base type's interface.
- **I**: Keep role interfaces small.
- **D**: Inject dependencies via constructor attributes.

## Documentation (POD)

- `=head1 NAME`, `=head1 SYNOPSIS`, `=head1 METHODS` for all public modules.
- `=head2 method_name($param)` for each public method.
- Document parameters, return values, exceptions.

## Error Handling

- `die` / `croak` for errors. `warn` / `carp` for warnings.
- `croak` (from `Carp`) for library code — reports from caller's perspective.
- Always check system calls: `open(...) or die "..."`.
- Structured exceptions: `Throwable` role (Moo/Moose) or custom classes. `Throwable::Error` is Moose-based — use only with Moose hierarchies.
- `Try::Tiny` for try/catch on older Perls. Built-in `try`/`catch` was experimental in 5.34, stabilised (no `experimental` warning) from 5.40.
- Lexical filehandles for auto-close on scope exit.

## Imports

- Order: pragmas, core modules, CPAN modules, project modules.
- One module per file. File path matches package name.
- Export sparingly — prefer OO interfaces.
- One module per concern: one HTTP client, one JSON library, one OO framework, one logging framework.

## Database / ACID

- DBI with `RaiseError => 1`, `AutoCommit => 0` for transactional code.
- Parameterised queries: `$sth->execute($email)` with placeholders. Never interpolate SQL.
- Rollback on failure, commit on success.
- Connection pools (`DBIx::Connector`). Disconnect or let `$dbh` go out of scope.
- Never interpolate or concatenate variables into SQL strings. Always use DBI placeholders (`?`).
- Use `$dbh->quote()` only as last resort — prefer placeholders.
- Validate and constrain input before it reaches the database layer.

## Testing

- Test2::V0 (modern) or Test::More (classic).
- `subtest 'descriptive name' => sub { ... };` structure.
- Test behaviour, not implementation.
- Test2::Tools::Mock or Test::MockObject for mocking at system boundaries.

## Performance & Idioms

- `qr//` for pre-compiled regexes. `/x` for readable patterns.
- Named captures: `(?<name>...)`.
- `map`, `grep`, `sort` for transformations.
- `List::Util` and `Scalar::Util` from core.
- Guard clauses for early returns.
- Understand scalar vs. list context.

## Patterns

- Factory: class method or standalone function.
- Builder: method chaining. Singleton: `state` variable or `MooseX::Singleton`.
- Strategy: inject coderef or role-consuming object.
- Observer: callbacks or event system.

## Defensive Programming & Input Validation

- Validate all external input at subroutine/module boundaries.
- Use `Params::Validate` or Moo/Moose type constraints for parameter validation.
- Validate string lengths and numeric ranges before processing.
- Use `Types::Standard` constraints: `Str`, `Int`, `Num`, `ArrayRef`, `HashRef`.
- Use `Scalar::Util::looks_like_number()` for numeric validation.
- Use taint mode (`-T`) for CGI/web scripts.
- Sanitize input used in file paths, system commands (`IPC::System::Simple`), and regex (`quotemeta`).
- Never use `eval` with unsanitized user input.
- Never pass user input to backticks, `system()`, or `open` pipe mode without sanitization.

## Project Structure

- `lib/` for modules, `t/` for tests, `script/` for executables.
- `cpanfile` for dependencies. `Carton` for installation.
- `Makefile.PL` or `Build.PL` with `PREREQ_PM`.

## Tooling

- Perl::Tidy (formatter), Perl::Critic (linter).
- prove (test runner), Test2::V0 (test framework).
- Devel::Cover (coverage), cpanfile + Carton (dependencies).

## Build Tools

- Traditional: Makefile.PL + `make install`.
- Modern: Dist::Zilla (dzil) for distribution building and release automation.
- Use cpanfile + Carton to lock and manage dependencies reproducibly.
- `carton exec` to run scripts in Carton environment.
- Use Makefile wrapper for common tasks (install, test, build, release).
- Docker for containerized Perl applications.

## SBOM Creation

- Use `cpanfile` and Carton for dependency management.
- Lock `cpanfile.snapshot` in VCS for reproducibility.
- Pin module versions: `requires 'Moose', '2.2128';`.
- Run CPAN security checks on dependencies.
- Verify license compliance for all modules.
- Gate CI/CD on security and license checks.
