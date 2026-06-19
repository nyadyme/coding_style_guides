# Perl 5 Coding Style Guidelines

A comprehensive guide rooted in `perlstyle`, *Perl Best Practices*
(Conway, 2005), *Modern Perl* (chromatic, 2015), Perl::Critic, and
established software engineering literature — notably *Design Patterns*
(Gamma et al., 1994) and *Clean Code* (Martin, 2008).

---

## Table of Contents

1. [Philosophy](#1-philosophy)
2. [Code Layout & Formatting](#2-code-layout--formatting)
3. [Naming Conventions](#3-naming-conventions)
4. [Subroutines](#4-subroutines)
5. [Object-Oriented Perl](#5-object-oriented-perl)
6. [Documentation & Comments](#6-documentation--comments)
7. [Error Handling](#7-error-handling)
8. [Modules & Imports](#8-modules--imports)
9. [Design Patterns in Perl](#9-design-patterns-in-perl)
10. [Testing](#10-testing)
11. [Database Access & ACID](#11-database-access--acid)
12. [Performance & Idiomatic Perl](#12-performance--idiomatic-perl)
13. [Defensive Programming & Input Validation](#13-defensive-programming--input-validation)
14. [Project Structure](#14-project-structure)
15. [Tooling](#15-tooling)
16. [Build Tools](#16-build-tools)
17. [SBOM Creation](#17-sbom-creation)
18. [References](#18-references)

---

## 1. Philosophy

### 1.1 Perl Mottos

> "There's more than one way to do it" (TMTOWTDI)
> "Easy things should be easy, and hard things should be possible."

While Perl allows great flexibility, **consistency within a project** is
paramount. Pick a style and stick to it.

### 1.2 Guiding Principles

| Principle | Source | Summary |
|---|---|---|
| **Use strict and warnings** | *Modern Perl*, `perlstyle` | `use strict; use warnings;` in every file. No exceptions. |
| **Readability over cleverness** | *Perl Best Practices*, *Clean Code* | Perl's expressiveness is a tool, not a weapon. |
| **Single Responsibility** | *Clean Code* Ch. 10, SOLID | Every module and subroutine should have one reason to change. |
| **YAGNI** | XP / *Clean Code* | Do not build for hypothetical future requirements. |
| **Composition over inheritance** | *Design Patterns* Ch. 1 | Prefer delegation and roles (Moose roles) over deep hierarchies. |

---

## 2. Code Layout & Formatting

### 2.1 Formatter

Use **Perl::Tidy** (`perltidy`) to enforce consistent formatting. Configure
via `.perltidyrc`:

```
-l=80
-i=4
-ci=4
-st
-se
-vt=2
-cti=0
-pt=1
-bt=1
-sbt=1
-bbt=1
-nsfs
-nolq
```

### 2.2 Indentation

- **4 spaces** per indentation level. Never tabs (configure `perltidy -et=4`
  to expand tabs).

### 2.3 Line Length

- **80 characters** maximum.

### 2.4 Blank Lines

- **Two blank lines** between subroutine definitions.
- **One blank line** between logical sections within a subroutine.
- **One blank line** after the `use` block before the first code.
- **No blank line** at the start or end of a block.
- **No trailing blank lines** at the end of a file.

### 2.5 Braces

K&R style (opening brace on same line):

```perl
if ($condition) {
    do_something();
} elsif ($other) {
    do_other();
} else {
    do_default();
}

for my $item (@items) {
    process($item);
}
```

### 2.6 Boilerplate

Every Perl file **must** start with:

```perl
use strict;
use warnings;
```

Or, for modern Perl (5.36+):

```perl
use v5.36;  # implies strict, warnings, say, etc.
```

Executable scripts need a shebang:

```perl
#!/usr/bin/env perl
use strict;
use warnings;
```

---

## 3. Naming Conventions

| Entity | Convention | Example |
|---|---|---|
| Package / Module | `PascalCase::Separated` | `My::App::UserService` |
| Subroutine | `snake_case` | `calculate_total()` |
| Variable (scalar) | `$snake_case` | `$total_count` |
| Variable (array) | `@snake_case` | `@user_names` |
| Variable (hash) | `%snake_case` | `%config_options` |
| Constant | `UPPER_SNAKE_CASE` | `use constant MAX_RETRIES => 3;` |
| Private sub/method | Leading `_` | `_validate_input()` |
| File | `PascalCase.pm` for modules | `UserService.pm` |
| Script | `kebab-case` or `snake_case` | `deploy-app.pl` |
| Boolean | Prefix `is_`, `has_`, `can_` | `$is_valid` |

### 3.1 Naming Guidance

- Meaningful, intention-revealing names. `$elapsed_seconds` beats `$s`.
- No single-letter variables except `$_`, loop indices, and temporary values.
- Sigils (`$`, `@`, `%`) already indicate type — no Hungarian notation.

---

## 4. Subroutines

### 4.1 Size and Focus

- Small (under 20 lines), one thing, one level of abstraction.
- Extract complex logic into named subroutines.

### 4.2 Parameters

Unpack `@_` explicitly at the top of every subroutine:

```perl
sub connect {
    my ($host, $port, %opts) = @_;
    my $timeout = $opts{timeout} // 30;
    my $retries = $opts{retries} // 3;
    # ...
}
```

- Use hash arguments for optional/named parameters.
- Validate arguments early.

For Perl 5.36+ with subroutine signatures:

```perl
use v5.36;

sub connect ($host, $port, %opts) {
    my $timeout = $opts{timeout} // 30;
    # ...
}
```

### 4.3 Return Values

- Return explicitly: `return $result;`.
- Return meaningful values. Return `undef` for absence, `die` for errors.
- Return list/array in list context, scalar in scalar context when appropriate.

### 4.4 Prototypes

Avoid prototypes except for emulating built-in syntax. They do not provide
type safety and confuse callers.

---

## 5. Object-Oriented Perl

### 5.1 Use Moose or Moo

Prefer **Moo** (lightweight) or **Moose** (full-featured) over manual
`bless`-based OO:

```perl
package User;
use Moo;
use Types::Standard qw(Str Int Bool);

has name      => (is => 'ro', isa => Str, required => 1);
has email     => (is => 'ro', isa => Str, required => 1);
has age       => (is => 'ro', isa => Int);
has is_active => (is => 'rw', isa => Bool, default => 1);

sub greet {
    my ($self) = @_;
    return "Hello, " . $self->name;
}

1;
```

### 5.2 Roles over Inheritance

Use **roles** (Moo::Role / Moose::Role) instead of deep inheritance:

```perl
package Loggable;
use Moo::Role;

requires 'logger';

sub log_info {
    my ($self, $message) = @_;
    $self->logger->info("[" . ref($self) . "] $message");
}

package UserService;
use Moo;
with 'Loggable';

has logger => (is => 'ro', required => 1);
```

### 5.3 Typed Data Passing

Pass structured, typed data between components — never raw hashrefs or
loosely typed data:

- Pass Moo/Moose objects, not raw hashrefs or arrayrefs, between subroutines
  and modules.
- Define clear data classes for data that crosses module boundaries.
- At system boundaries (API responses, config files, CLI args), parse into
  typed objects immediately.
- Use `Types::Standard` constraints to enforce data shapes.
- Return objects, not lists of loosely related values.

```perl
# Bad — raw hashref
sub process {
    my ($data) = @_;  # $data is an untyped hashref
    my $name = $data->{name};  # no validation, no guarantees
}

# Good — typed object
package UserPayload;
use Moo;
use Types::Standard qw(Str Int);

has name  => (is => 'ro', isa => Str, required => 1);
has email => (is => 'ro', isa => Str, required => 1);
has age   => (is => 'ro', isa => Int);

package main;

sub process {
    my ($payload) = @_;  # $payload is a UserPayload object
    my $name = $payload->name;  # validated, typed
}
```

### 5.4 SOLID in Perl

| Principle | Perl Application |
|---|---|
| **S** — Single Responsibility | One module per concern. |
| **O** — Open/Closed | Extend via roles and composition. |
| **L** — Liskov Substitution | Subtypes must honour the base type's interface. |
| **I** — Interface Segregation | Keep role interfaces small. |
| **D** — Dependency Inversion | Inject dependencies via constructor attributes. |

---

## 6. Documentation & Comments

### 6.1 POD (Plain Old Documentation)

Document all public modules and methods with POD:

```perl
=head1 NAME

My::App::UserService - Manage user accounts

=head1 SYNOPSIS

    my $service = My::App::UserService->new(db => $db);
    my $user = $service->find_user(42);

=head1 METHODS

=head2 find_user($id)

Retrieve a user by their unique identifier.

=over 4

=item * C<$id> - The user's integer ID.

=back

Returns a L<User> object or C<undef> if not found.

Throws L<My::App::Error> on database failure.

=cut
```

### 6.2 Comments

- Comments explain **why**, not what.
- Never commit commented-out code.
- Use `# TODO:` and `# FIXME:` sparingly.

---

## 7. Error Handling

### 7.1 Principles

- Use `die` / `croak` for errors. Use `warn` / `carp` for warnings.
- `croak` (from `Carp`) reports from the caller's perspective — preferred
  for library code.
- Always check the return value of system calls: `open(...) or die "..."`.

### 7.2 Structured Exceptions

Use exception objects (e.g., `Throwable`, custom classes):

```perl
package My::App::Error;
use Moo;
with 'Throwable';

has message => (is => 'ro', required => 1);
has code    => (is => 'ro', default => sub { 'UNKNOWN' });

package My::App::NotFoundError;
use Moo;
extends 'My::App::Error';
has '+code' => (default => sub { 'NOT_FOUND' });
```

Note: `Throwable` is a role consumable by Moo and Moose classes.
`Throwable::Error` is a Moose-based class — use it only with Moose-based
class hierarchies.

### 7.3 Try/Catch

Use `Try::Tiny`, or the built-in `try`/`catch` block (experimental in Perl
5.34; stabilised — no `experimental` warning — from Perl 5.40):

```perl
use Try::Tiny;

try {
    my $result = risky_operation();
} catch {
    warn "Operation failed: $_";
};
```

### 7.4 Resource Cleanup

Use lexical filehandles (auto-close on scope exit):

```perl
{
    open my $fh, '<', $path or die "Cannot open $path: $!";
    while (my $line = <$fh>) {
        process($line);
    }
}  # $fh closed here
```

---

## 8. Modules & Imports

### 8.1 Module Design

- One module per file. File path matches package name:
  `My/App/UserService.pm` for `My::App::UserService`.
- End every module with `1;` (true return value).
- Export sparingly — prefer OO interfaces.

### 8.2 Import Ordering

1. Pragmas (`use strict`, `use warnings`, `use v5.36`)
2. Core modules (`use File::Basename`, `use Carp`)
3. CPAN modules (`use Moo`, `use DBI`)
4. Project modules

### 8.3 One Module per Concern

| Overlapping modules | Pick one |
|---|---|
| `LWP` / `HTTP::Tiny` / `Mojo::UserAgent` | One HTTP client |
| `JSON::XS` / `JSON::PP` / `Cpanel::JSON::XS` | `JSON::MaybeXS` (auto-selects) |
| `Moose` / `Moo` / `Mouse` | One OO framework per project |
| `DateTime` / `Time::Piece` | One time library |
| `Log::Log4perl` / `Log::Any` / `Log::Dispatch` | One logging framework |

---

## 9. Design Patterns in Perl

### 9.1 Creational

- **Factory**: class method or standalone function.
- **Builder**: method chaining. **Singleton**: module-level `state` variable
  or `MooseX::Singleton`.

### 9.2 Structural

- **Decorator**: delegation wrapper.
- **Adapter**: wraps incompatible interface behind a role.

### 9.3 Behavioural

- **Strategy**: inject coderef or role-consuming object.
- **Observer**: callbacks or event system.
- **Template Method**: base class with abstract methods (via `die` or
  `requires` in roles).

---

## 10. Testing

### 10.1 Principles

- Use **Test2::V0** (modern) or **Test::More** (classic).
- Test behaviour, not implementation. Testing pyramid.

### 10.2 Structure

```perl
use Test2::V0;

subtest 'withdraw with insufficient funds' => sub {
    my $account = Account->new(balance => 100);

    my $exception = dies { $account->withdraw(200) };

    like $exception, qr/insufficient funds/i;
};

done_testing;
```

### 10.3 Mocking

- Use **Test2::Tools::Mock** or **Test::MockObject**.
- Mock at system boundaries (DBI, HTTP, filesystem).

---

## 11. Database Access & ACID

### 11.1 Use DBI with Transactions

```perl
my $dbh = DBI->connect($dsn, $user, $pass, {
    RaiseError => 1,
    AutoCommit => 0,
});

eval {
    $dbh->do("UPDATE accounts SET balance = balance - ? WHERE id = ?",
        undef, $amount, $from_id);
    $dbh->do("UPDATE accounts SET balance = balance + ? WHERE id = ?",
        undef, $amount, $to_id);
    $dbh->commit;
};
if ($@) {
    $dbh->rollback;
    die "Transfer failed: $@";
}
```

### 11.2 Parameterised Queries

```perl
# Yes — placeholders
my $sth = $dbh->prepare("SELECT * FROM users WHERE email = ?");
$sth->execute($email);

# No — interpolation (SQL injection)
$dbh->do("SELECT * FROM users WHERE email = '$email'");
```

### 11.3 SQL Injection Protection

Never build SQL queries by interpolating variables. Always use DBI
placeholders:

```perl
# Bad — variable interpolation (SQL injection vulnerability)
my $sth = $dbh->prepare("SELECT * FROM users WHERE name = '$name'");
$sth->execute();

# Good — placeholders with bound parameters
my $sth = $dbh->prepare("SELECT * FROM users WHERE name = ?");
$sth->execute($name);
```

- Always use DBI placeholders (`?`) with `$sth->execute(@params)`.
- Never interpolate variables into SQL strings.
- Use `$dbh->quote()` only as last resort — prefer placeholders.
- Validate and constrain input before it reaches the database layer.

### 11.4 Connection Lifecycle

- Set `RaiseError => 1` and `AutoCommit => 0` for transactional code.
- Use connection pools (e.g., `DBIx::Connector`).
- Always disconnect or let `$dbh` go out of scope.

---

## 12. Performance & Idiomatic Perl

### 12.1 Regular Expressions

- Use `qr//` for pre-compiled regexes.
- Use `/x` for readable multi-line patterns.
- Use named captures: `(?<name>...)`.

### 12.2 Data Structures

- Use `map`, `grep`, `sort` for transformations.
- Use hash slices and array slices.
- Use `List::Util` and `Scalar::Util` from core.

### 12.3 Context Awareness

Understand scalar vs. list context. Use `wantarray()` when writing
context-sensitive functions.

### 12.4 Guard Clauses

```perl
sub process_order {
    my ($self, $order) = @_;
    die "Order cancelled" if $order->is_cancelled;
    return $EMPTY_RECEIPT   if $order->is_empty;
    return $self->_build_receipt($order);
}
```

---

## 13. Defensive Programming & Input Validation

### 13.1 Validate External Input at Boundaries

Validate all external input at subroutine and module boundaries. Never
trust data from users, APIs, files, or the network.

### 13.2 Parameter Validation

Use `Params::Validate` or Moo/Moose type constraints for parameter
validation:

```perl
use Types::Standard qw(Str Int Num ArrayRef HashRef);

has name => (is => 'ro', isa => Str, required => 1);
has port => (is => 'ro', isa => Int);
```

### 13.3 Value Constraints

Validate string lengths, numeric ranges, and data shapes:

```perl
die "Name too long" if length($name) > 255;
die "Invalid port"  unless $port >= 1 && $port <= 65535;
```

- Use `Types::Standard` constraints: `Str`, `Int`, `Num`, `ArrayRef`,
  `HashRef`.
- Use `Scalar::Util::looks_like_number()` for numeric validation.
- Check array bounds before indexing.

### 13.4 Taint Mode

Use taint mode (`-T`) for CGI/web scripts to track untrusted data:

```perl
#!/usr/bin/perl -T
use strict;
use warnings;
```

Note: `-T` must appear on the shebang line of the interpreter directly.
`#!/usr/bin/env perl -T` does not work portably because `env` does not pass
flags reliably across platforms — and taint mode must be enabled at startup.

### 13.5 Dangerous Operations

- Sanitize input used in file paths — validate against path traversal
  (`..`).
- Use `IPC::System::Simple` for safe system commands.
- Use `quotemeta` to escape input used in regular expressions.
- Never use `eval` with unsanitized user input.
- Never pass user input to backticks, `system()`, or `open` pipe mode
  without sanitization.

```perl
# Bad — user input in system command
system("ls $user_dir");

# Good — use list form of system
system('ls', $user_dir);

# Good — use IPC::System::Simple
use IPC::System::Simple qw(systemx);
systemx('ls', $user_dir);
```

---

## 14. Project Structure

### 14.1 Recommended Layout

```
My-App/
    Makefile.PL / dist.ini / cpanfile
    lib/
        My/
            App.pm
            App/
                UserService.pm
                Error.pm
    t/
        01-user-service.t
    script/
        my-app.pl
    Changes
    README.md
```

### 14.2 Dependency Management

- Use **cpanfile** for dependencies. **Carton** for installation.
- Or `Makefile.PL` / `Build.PL` with `PREREQ_PM`.

---

## 15. Tooling

| Purpose | Tool | Notes |
|---|---|---|
| Formatter | **Perl::Tidy** | Consistent formatting |
| Linter | **Perl::Critic** | Enforce *Perl Best Practices* |
| Test runner | **prove** | `prove -lr t/` |
| Test framework | **Test2::V0** / **Test::More** | Modern testing |
| Coverage | **Devel::Cover** | Statement and branch coverage |
| Dependencies | **cpanfile** + **Carton** | Reproducible installs |
| Debugger | **perl -d** / **Devel::REPL** | Interactive debugging |

---

## 16. Build Tools

### 16.1 Makefile.PL (Traditional)

Perl's traditional build system:

```perl
# Makefile.PL
use ExtUtils::MakeMaker;

WriteMakefile(
    NAME => 'MyApp::Module',
    VERSION_FROM => 'lib/MyApp/Module.pm',
    PREREQ_PM => {
        'Moose' => 2.21,
        'DBIx::Class' => 0.08,
    },
    test => { TESTS => 't/*.t' },
);
```

Build and test:
```bash
perl Makefile.PL
make
make test
make install
```

### 16.2 Module::Build (Alternative)

More flexible build system:

```perl
# Build.PL
use Module::Build;

my $build = Module::Build->new(
    module_name => 'MyApp::Module',
    requires => { 'Moose' => 2.21 },
    test_files => 't/',
);
$build->create_build_script;
```

### 16.3 Dist::Zilla (Modern & Recommended)

Powerful distribution building and release automation:

```perl
# dist.ini
name    = MyApp-Module
author  = Author Name <author@example.com>
license = MIT
copyright_holder = Author Name

[GatherDir]
[PruneCruft]
[ManifestSkip]

[@Basic]
[AutoVersion]
[PodWeaver]

[Prereqs]
Moose = 2.21
DBIx::Class = 0.08
```

Build and release:
```bash
dzil build  # Build distribution
dzil test   # Test all Perl versions with Perlbrew
dzil release  # Release to CPAN
```

### 16.4 cpanfile and Carton

Declare and lock dependencies:

```perl
# cpanfile
requires 'Moose', '2.21';
requires 'DBIx::Class', '0.08';

on development => sub {
    requires 'Test::More', '1.30';
};
```

Lock and install:
```bash
carton install  # Creates cpanfile.snapshot
carton exec -- perl script.pl  # Run in Carton environment
```

### 16.5 Make or Makefile Wrapper

Simple Makefile for running Carton/Perl tasks:

```makefile
.PHONY: install test build release

install:
	carton install

test:
	carton exec -- prove -l t/

build:
	carton exec -- dzil build

release:
	carton exec -- dzil release
```

### 16.6 Docker for Perl Applications

```dockerfile
FROM perl:5.38
WORKDIR /app
COPY cpanfile* ./
RUN cpanm --installdeps .
COPY . .
CMD ["perl", "script.pl"]
```

---

## 17. SBOM Creation

### 17.1 What is an SBOM?

A Software Bill of Materials lists all Perl modules (dependencies) and their versions. Critical for reproducibility, vulnerability tracking, and license compliance.

### 17.2 Dependency Declaration with cpanfile

Modern Perl projects use `cpanfile` to declare dependencies:
```perl
requires 'Moose', '2.2128';
requires 'DBIx::Class', '0.082841';
requires 'JSON::XS', '4.03';

test_requires 'Test::More', '1.302195';
```

Install with Carton (locks versions):
```bash
carton install  # Creates cpanfile.snapshot
```

### 17.3 SBOM Generation

**From cpanfile.snapshot**:

The Carton snapshot is the authoritative pinned dependency manifest. Parse
it directly to emit a CycloneDX/SPDX document:

```bash
# Parse the snapshot and emit JSON via Module::CPANfile + Carton::Snapshot
perl -MCarton::Snapshot -e '
    my $s = Carton::Snapshot->new(path => "cpanfile.snapshot");
    $s->load;
    for my $dist ($s->distributions) {
        printf "%s %s\n", $dist->name, $dist->version;
    }
'
```

`Module::Manifest` parses a distribution's `MANIFEST` file — it lists files
in the distribution, not dependencies. Do not use it for SBOM input.

**CycloneDX tooling**: there is no first-party CPAN module that emits
CycloneDX. Use a generic tool such as `cdxgen` (which understands cpanfile
and cpanfile.snapshot) or write a custom converter.

### 17.4 Vulnerability Scanning

**Using `CPAN::Audit`** (queries the CPAN Security Advisory Database):

```bash
# CLI form — the canonical interface
cpan-audit installed                 # audit modules in @INC
cpan-audit dependencies cpanfile     # audit a cpanfile
```

Or use external tools like Snyk for continuous monitoring.

### 17.5 License Compliance

License metadata lives in each distribution's `META.json` / `META.yml`, not
in the `.pm` source. Read it via `CPAN::Meta`:

```perl
use CPAN::Meta;
my $meta = CPAN::Meta->load_file('META.json');
my @licenses = @{ $meta->license };  # e.g. ['perl_5'], ['mit']
```

`Module::Metadata` only inspects `package`/`$VERSION` declarations in a
`.pm` file — it does not return license information. Use `Software::License`
to render full license text when generating distribution metadata.

Verify all licenses in dependencies are acceptable (MIT, Apache-2.0, GPL-compatible, etc.).

### 17.6 Integration into CI/CD

- Lock `cpanfile.snapshot` in VCS with Carton (required for reproducibility)
- Run CPAN security checks in CI
- Verify license compliance
- Generate SBOM on release
- Monitor CPAN security advisories
- Store SBOM and audit reports with release

---

## 18. References

### Official Documentation

| Resource | URL |
|---|---|
| perlstyle | https://perldoc.perl.org/perlstyle |
| perlsub | https://perldoc.perl.org/perlsub |
| perlootut | https://perldoc.perl.org/perlootut |
| Perl documentation | https://perldoc.perl.org |

### Books

| Book | Authors | Key Takeaways |
|---|---|---|
| *Perl Best Practices* | Damian Conway (2005) | Comprehensive style and practice guide — the foundation for Perl::Critic. |
| *Modern Perl* | chromatic (2015) | Idiomatic modern Perl: Moose, testing, best practices. |
| *Programming Perl* | Wall, Christiansen, Orwant (2012) | The "Camel Book" — definitive reference. |
| *Higher-Order Perl* | Mark Jason Dominus (2005) | Functional programming techniques in Perl. |
| *Design Patterns* | Gamma et al. (1994) | Composition over inheritance. |
| *Clean Code* | Robert C. Martin (2008) | Small functions, meaningful names, SRP. |
