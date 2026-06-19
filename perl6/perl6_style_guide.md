# Raku (Perl 6) Coding Style Guidelines

A comprehensive guide rooted in the official Raku documentation, *Think
Raku* (Moreno, 2020), *Perl 6 Deep Dive* (Szucs, 2017), and established
software engineering literature — notably *Design Patterns* (Gamma et al.,
1994) and *Clean Code* (Martin, 2008).

---

## Table of Contents

1. [Philosophy](#1-philosophy)
2. [Code Layout & Formatting](#2-code-layout--formatting)
3. [Naming Conventions](#3-naming-conventions)
4. [Subroutines & Methods](#4-subroutines--methods)
5. [Object-Oriented Raku](#5-object-oriented-raku)
6. [Documentation & Comments](#6-documentation--comments)
7. [Error Handling](#7-error-handling)
8. [Modules & Imports](#8-modules--imports)
9. [Design Patterns in Raku](#9-design-patterns-in-raku)
10. [Testing](#10-testing)
11. [Database Access & ACID](#11-database-access--acid)
12. [Concurrency](#12-concurrency)
13. [Type System & Functional Features](#13-type-system--functional-features)
14. [Performance & Idiomatic Raku](#14-performance--idiomatic-raku)
15. [Defensive Programming & Input Validation](#15-defensive-programming--input-validation)
16. [Project Structure](#16-project-structure)
17. [Tooling](#17-tooling)
18. [Build Tools](#18-build-tools)
19. [SBOM Creation](#19-sbom-creation)
20. [References](#20-references)

---

## 1. Philosophy

### 1.1 Raku's Core Values

> "Raku: Second best at everything."

Raku is a multi-paradigm language designed for expressiveness, gradual
typing, powerful grammars, and built-in concurrency. It is a distinct
language from Perl 5 — not merely a version bump.

### 1.2 Guiding Principles

| Principle | Source | Summary |
|---|---|---|
| **TMTOWTDI** | Raku tradition | Multiple solutions exist; pick the clearest for your context. |
| **Gradual typing** | Raku design | Add type constraints where they improve safety and documentation. |
| **Concurrency by default** | Raku design | Use promises, channels, and supplies — no manual thread management. |
| **Grammars for parsing** | Raku design | Use grammars instead of complex regex chains. |
| **Composition over inheritance** | *Design Patterns* Ch. 1 | Prefer roles over deep class hierarchies. |
| **Single Responsibility** | *Clean Code* Ch. 10, SOLID | Every class and subroutine should have one reason to change. |
| **YAGNI** | XP / *Clean Code* | Do not build for hypothetical future requirements. |

---

## 2. Code Layout & Formatting

### 2.1 Indentation

- **4 spaces** per indentation level. Never tabs.

### 2.2 Line Length

- **100 characters** recommended maximum.

### 2.3 Blank Lines

- **Two blank lines** between subroutine/method definitions at the top level.
- **One blank line** between methods within a class.
- **One blank line** between logical sections within a subroutine.
- **One blank line** after `use` statements before the first code.
- **No blank line** at the start or end of a block.
- **No trailing blank lines** at the end of a file.

### 2.4 Braces

Opening brace on the same line:

```raku
if $condition {
    do-something();
} elsif $other {
    do-other();
} else {
    do-default();
}

for @items -> $item {
    process($item);
}
```

### 2.5 Long Lines

Break long lines after operators or commas:

```raku
my $result = compute-first-part()
    + compute-second-part()
    + compute-third-part();

my @processed = @data
    .grep(*.is-valid)
    .map(*.transform)
    .sort;
```

---

## 3. Naming Conventions

| Entity | Convention | Example |
|---|---|---|
| Module / Class | `PascalCase::Separated` | `My::App::UserService` |
| Role | `PascalCase` | `Printable`, `Serializable` |
| Subroutine | `kebab-case` | `calculate-total()` |
| Method | `kebab-case` | `$user.full-name` |
| Variable (scalar) | `$kebab-case` | `$total-count` |
| Variable (array) | `@kebab-case` | `@user-names` |
| Variable (hash) | `%kebab-case` | `%config-options` |
| Constant | `kebab-case` with `constant` | `constant max-retries = 3;` |
| Private method | Leading `!` (twigil) | `method !validate-input` |
| Attribute | `$.public` / `$!private` | `has $.name`, `has $!cache` |
| File | `PascalCase.rakumod` for modules | `UserService.rakumod` |
| Script | `kebab-case` | `deploy-app.raku` |
| Boolean | Prefix `is-`, `has-`, `can-` | `$is-valid` |
| Grammar | `PascalCase` | `JSONParser` |
| Token / Rule | `kebab-case` | `token identifier` |

### 3.1 Naming Guidance

- Meaningful, intention-revealing names. `$elapsed-seconds` beats `$s`.
- Raku's kebab-case is idiomatic — use it consistently.
- Sigils (`$`, `@`, `%`) already indicate container type.
- Twigils (`!`, `.`, `^`, `:`) have specific meanings — use them correctly.

---

## 4. Subroutines & Methods

### 4.1 Size and Focus

- Small (under 20 lines), one thing, one level of abstraction.

### 4.2 Signatures

Raku has built-in parameter declarations in signatures:

```raku
sub connect(Str $host, Int $port, Int :$timeout = 30, Int :$retries = 3) {
    # ...
}
```

- Use type constraints in signatures.
- Use named parameters (`:$param`) for optional arguments.
- Use `where` clauses for value constraints.
- Use multi dispatch for polymorphic behaviour.

### 4.3 Multi Dispatch

```raku
multi sub process(Int $n) { say "Integer: $n" }
multi sub process(Str $s) { say "String: $s" }
multi sub process(Rat $r) { say "Rational: $r" }
```

### 4.4 Return Values

- Use explicit return types in signatures: `sub find-user(Int $id --> User) { ... }`.
- Return `Nil` for absence. Use `fail` for soft failures. Use `die` for
  hard errors.
- Return type constraints are enforced at runtime.

### 4.5 Blocks and Pointy Blocks

```raku
# Pointy block
for @items -> $item {
    process($item);
}

# With signature
my &processor = -> Str $name, Int $age {
    say "$name is $age years old";
};
```

---

## 5. Object-Oriented Raku

### 5.1 Class Definition

Raku has built-in, well-designed OO:

```raku
class User {
    has Str $.name is required;
    has Str $.email is required;
    has Int $.age;
    has Bool $.active = True;

    method greet(--> Str) {
        "Hello, $.name"
    }
}
```

- `$.attr` creates public accessor. `$!attr` is private.
- `is required` enforces mandatory attributes.
- Use `BUILD` or `TWEAK` submethods for custom construction logic.

### 5.2 Roles over Inheritance

Roles are Raku's primary composition mechanism:

```raku
role Loggable {
    has $.logger is required;

    method log-info(Str $message) {
        $.logger.info("[{self.^name}] $message");
    }
}

class UserService does Loggable {
    # $.logger is supplied by the Loggable role — do not redeclare,
    # or composition will fail with an attribute conflict.

    method find-user(Int $id --> User) {
        self.log-info("Looking up user $id");
        # ...
    }
}
```

- Use `does` for role composition.
- Roles can require methods and attributes.
- Roles detect conflicts at composition time.

### 5.3 Inheritance (When Needed)

```raku
class Animal {
    has Str $.name;
    method speak(--> Str) { ... }  # Abstract (stub)
}

class Dog is Animal {
    method speak(--> Str) { "Woof!" }
}
```

### 5.4 Typed Data Passing

Pass structured, typed data between components — never raw hashes or
untyped collections:

- Pass typed objects, not raw hashes or arrays, between methods and modules.
- Use type constraints in signatures to enforce data contracts.
- At system boundaries (API responses, config files, CLI args), parse into
  typed objects immediately.
- Use `subset` types to constrain value domains.
- Return typed objects with explicit return type annotations (`-->`).

```raku
# Bad — raw hash
sub process(%data) { ... }

# Good — typed object
class UserPayload {
    has Str $.name is required;
    has Str $.email is required;
    has Int $.age;
}

sub process(UserPayload $payload --> ProcessResult) {
    # validated, typed access
    my $name = $payload.name;
    ...
}
```

### 5.5 SOLID in Raku

| Principle | Raku Application |
|---|---|
| **S** — Single Responsibility | One class per concern. |
| **O** — Open/Closed | Extend via roles, multi dispatch, and augment. |
| **L** — Liskov Substitution | Subtypes must honour the base type's interface. |
| **I** — Interface Segregation | Keep roles small and focused. |
| **D** — Dependency Inversion | Inject dependencies via constructor attributes. |

---

## 6. Documentation & Comments

### 6.1 Declarator Blocks

Raku uses declarator blocks (`#|` before, `#=` after) for documentation:

```raku
#| Retrieve a user by their unique identifier.
#| Looks up the user in the cache first, then falls back to the database.
method find-user(
    Int $id,          #= The user's unique integer ID
    Bool :$include-deleted = False  #= Include soft-deleted users
    --> User
) {
    # ...
}
```

- `#|` documents the next declaration.
- `#=` documents the preceding declaration.
- Documentation is accessible at runtime via `.WHY`.

### 6.2 Pod6

For longer documentation, use Pod6:

```raku
=begin pod

=head1 NAME

My::App::UserService — Manage user accounts

=head1 SYNOPSIS

    my $service = My::App::UserService.new(db => $db);
    my $user = $service.find-user(42);

=head1 DESCRIPTION

Provides CRUD operations for user accounts.

=end pod
```

### 6.3 Comments

- Comments explain **why**, not what.
- Never commit commented-out code.
- Use `# TODO:` and `# FIXME:` sparingly.

---

## 7. Error Handling

### 7.1 Exceptions

Raku has typed exceptions:

```raku
class X::App::NotFound is Exception {
    has Str $.entity is required;
    has $.id is required;

    method message(--> Str) {
        "$.entity with id $.id not found"
    }
}
```

### 7.2 die, fail, and try

```raku
# Hard failure — immediately throws
die X::App::NotFound.new(entity => 'User', id => 42);

# Soft failure — returns a Failure object
sub find-user(Int $id --> User) {
    fail X::App::NotFound.new(entity => 'User', id => $id)
        unless %users{$id};
    %users{$id}
}

# Catching
try {
    my $user = find-user(99);
    CATCH {
        when X::App::NotFound { warn .message }
        default { .rethrow }
    }
}
```

- Use `die` for unrecoverable errors.
- Use `fail` for recoverable failures (lazy exceptions).
- Use `CATCH` phasers for structured error handling.
- Use typed exceptions for domain errors.

### 7.3 Resource Cleanup

Use `LEAVE` phaser for cleanup:

```raku
sub process-file(Str $path) {
    my $fh = open $path, :r;
    LEAVE $fh.close;
    for $fh.lines -> $line {
        process($line);
    }
}
```

---

## 8. Modules & Imports

### 8.1 Module Design

```raku
unit module My::App::UserService;

# Module-level code
```

Or block form:

```raku
module My::App::UserService {
    # ...
}
```

- File path matches module name: `My/App/UserService.rakumod`.
- Use `is export` trait selectively — don't export everything.

### 8.2 Import Ordering

1. Pragmas (`use v6.d;`)
2. Core modules
3. Ecosystem modules (from Zef)
4. Project modules

### 8.3 One Module per Concern

| Overlapping modules | Pick one |
|---|---|
| `HTTP::Tiny` / `Cro::HTTP` / `LWP::Simple` | One HTTP client |
| `JSON::Fast` / `JSON::Tiny` / `JSON::Class` | One JSON library |
| `DBIish` / `Red` | One database approach |
| `Log::Async` / custom logging | One logging framework |

---

## 9. Design Patterns in Raku

### 9.1 Creational

- **Factory**: multi subs or class methods.
- **Builder**: method chaining or named parameters.
- **Singleton**: module-level `my` variable with lazy initialisation.

### 9.2 Structural

- **Decorator**: role composition at runtime (`but` keyword).
- **Adapter**: role that wraps an incompatible interface.
- **Proxy**: `Proxy` class for custom container behaviour.

### 9.3 Behavioural

- **Strategy**: pass callables or role-consuming objects.
- **Observer**: supplies and taps.
- **Template Method**: roles with stub methods.
- **Visitor**: multi dispatch.

### 9.4 Runtime Mixins

```raku
role Timestamped {
    has DateTime $.created-at = DateTime.now;
}

my $user = User.new(name => 'Ada');
my $stamped = $user but Timestamped;
say $stamped.created-at;
```

---

## 10. Testing

### 10.1 Principles

- Use **Test** (built-in) or **Test::Async**.
- Test behaviour, not implementation. Testing pyramid.

### 10.2 Structure

```raku
use Test;

plan 3;

subtest 'withdraw with insufficient funds' => {
    my $account = Account.new(balance => 100);

    dies-ok { $account.withdraw(200) },
        'throws on insufficient funds';

    like $!, /insufficient/, 'error message mentions insufficient';
}

subtest 'successful withdrawal' => {
    my $account = Account.new(balance => 100);
    $account.withdraw(50);
    is $account.balance, 50, 'balance reduced correctly';
}

done-testing;
```

### 10.3 Test Functions

| Function | Purpose |
|---|---|
| `ok`, `nok` | Boolean truth/falsity |
| `is`, `isnt` | Equality comparison |
| `cmp-ok` | Comparison with operator |
| `like`, `unlike` | Regex matching |
| `dies-ok`, `lives-ok` | Exception testing |
| `throws-like` | Typed exception testing |
| `isa-ok` | Type checking |
| `subtest` | Grouped tests |

### 10.4 Mocking

Mock at system boundaries. Use role-based dependency injection to supply
test doubles.

---

## 11. Database Access & ACID

### 11.1 Use DBIish

```raku
use DBIish;

my $dbh = DBIish.connect('Pg',
    host     => 'localhost',
    database => 'myapp',
    user     => $user,
    password => $pass,
);
```

### 11.2 Transactions

```raku
$dbh.do('BEGIN');
{
    CATCH {
        default {
            $dbh.do('ROLLBACK');
            .rethrow;
        }
    }
    $dbh.do('UPDATE accounts SET balance = balance - ? WHERE id = ?',
        $amount, $from-id);
    $dbh.do('UPDATE accounts SET balance = balance + ? WHERE id = ?',
        $amount, $to-id);
    $dbh.do('COMMIT');
}
```

A bare block with a `CATCH` phaser propagates the exception after rollback.
Wrapping in `try` would suppress the rethrow — only use `try` when the
caller genuinely wants to inspect `$!` rather than let the exception bubble.

### 11.3 Parameterised Queries

```raku
# Yes — placeholders
my $sth = $dbh.prepare('SELECT * FROM users WHERE email = ?');
$sth.execute($email);

# No — interpolation (SQL injection)
$dbh.do("SELECT * FROM users WHERE email = '$email'");
```

### 11.4 SQL Injection Protection

Never build SQL queries by interpolating variables. Always use
parameterised queries with placeholders:

```raku
# Bad — variable interpolation (SQL injection vulnerability)
$dbh.do("SELECT * FROM users WHERE email = '$email'");

# Good — parameterised query with placeholder
my $sth = $dbh.prepare('SELECT * FROM users WHERE email = ?');
$sth.execute($email);
```

- Always use parameterised queries with `?` placeholders via DBIish.
- Never interpolate variables into SQL strings.
- Validate and constrain input before it reaches the database layer.

---

## 12. Concurrency

### 12.1 Promises

```raku
my $promise = start {
    expensive-computation()
};

my $result = await $promise;
```

### 12.2 Channels

```raku
my $channel = Channel.new;

start {
    for 1..10 -> $i {
        $channel.send($i);
    }
    $channel.close;
}

react {
    whenever $channel -> $item {
        say "Got: $item";
    }
}
```

### 12.3 Supplies (Reactive Streams)

```raku
my $supply = supply {
    for 1..10 -> $i {
        emit $i;
    }
}

$supply.tap(-> $value { say $value });
```

### 12.4 Hyper and Race

```raku
# Parallel map (unordered)
my @results = @data.race(:batch(100), :degree(4)).map(&process);

# Parallel map (ordered)
my @results = @data.hyper(:batch(100), :degree(4)).map(&process);
```

### 12.5 Concurrency Guidelines

- Use `start` blocks and `await` for async work.
- Use `react`/`whenever` for event-driven code.
- Use `hyper`/`race` for data parallelism.
- Never share mutable state — use channels or supplies.
- Use `Lock::Async` if shared state is unavoidable.

---

## 13. Type System & Functional Features

### 13.1 Gradual Typing

```raku
# Untyped (dynamic)
my $name = "Ada";

# Typed
my Str $name = "Ada";
my Int @numbers = 1, 2, 3;
my Str %config = port => "8080", host => "localhost";
```

- Add types where they improve clarity and catch bugs.
- Use `subset` for constrained types.

### 13.2 Subsets (Refined Types)

```raku
subset PositiveInt of Int where * > 0;
subset Email of Str where /^ \S+ '@' \S+ '.' \S+ $/;
subset NonEmptyStr of Str where *.chars > 0;

sub create-user(NonEmptyStr $name, Email $email, PositiveInt $age) {
    # ...
}
```

### 13.3 Junctions

```raku
if $x == any(1, 2, 3) { say "Match!" }
if $x == none(4, 5, 6) { say "Not in set" }
```

### 13.4 Functional Features

```raku
# Function composition
my &process = &validate o &transform o &normalize;

# Chaining with feed operators
@data ==> grep(*.is-valid)
     ==> map(*.transform)
     ==> sort()
     ==> my @result;

# Reduction
my $sum = [+] @numbers;
my $product = [*] @numbers;
```

### 13.5 Grammars

```raku
grammar INIFile {
    token TOP     { <section>* }
    token section { '[' <heading> ']' \n <keyval>* }
    token heading { \w+ }
    token keyval  { <key> '=' <value> \n }
    token key     { \w+ }
    token value   { \N+ }
}

class INIActions {
    method TOP($/)     { make $<section>».made.hash }
    method section($/) { make ~$<heading> => $<keyval>».made.hash }
    method keyval($/)  { make ~$<key>     => ~$<value> }
}

my $match = INIFile.parse($text, actions => INIActions.new);
my %config = $match.made;
```

`make` attaches an AST node to the current `Match`; `.made` retrieves it.
Use `~$<capture>` to coerce a `Match` to `Str` — `.Str` works too but `~`
is idiomatic. Pass the actions object via the named argument `:actions` to
`.parse` / `.parsefile`.

---

## 14. Performance & Idiomatic Raku

### 14.1 Idiomatic Constructs

- Use `given`/`when` for smart matching (switch).
- Use `gather`/`take` for lazy list generation.
- Use `...` (sequence operator) for series.
- Use `»` (hyper operator) for element-wise operations.

### 14.2 Lazy Evaluation

```raku
# Infinite sequence
my @fibs = 0, 1, * + * ... *;
say @fibs[10];  # 55

# Gather/take
my @evens = gather {
    for 0..* -> $n {
        take $n if $n %% 2;
    }
}
```

### 14.3 Guard Clauses

```raku
method process-order(Order $order --> Receipt) {
    die "Order cancelled" if $order.is-cancelled;
    return Receipt.empty   if $order.is-empty;
    self!build-receipt($order)
}
```

### 14.4 Smart Matching

```raku
given $value {
    when Int    { say "Integer" }
    when Str    { say "String" }
    when /foo/  { say "Contains foo" }
    when 1..10  { say "1 to 10" }
    default     { say "Something else" }
}
```

---

## 15. Defensive Programming & Input Validation

### 15.1 Validate External Input at Boundaries

Validate all external input at method and subroutine boundaries. Never
trust data from users, APIs, files, or the network.

### 15.2 Type Constraints in Signatures

Use type constraints and `where` clauses to enforce validation directly
in signatures:

```raku
sub process(Str $name where *.chars <= 255, Int $port where 1..65535) {
    # $name and $port are guaranteed valid
    ...
}
```

### 15.3 Subset Types for Reusable Validation

Use `subset` types to define reusable, self-documenting constraints:

```raku
subset Port of Int where 1..65535;
subset ShortName of Str where *.chars <= 255;
subset Email of Str where /^ \S+ '@' \S+ '.' \S+ $/;

sub create-user(ShortName $name, Email $email, Port :$port = 8080) {
    ...
}
```

### 15.4 Multi Dispatch for Safe Input Handling

Use multi dispatch for handling different input types safely:

```raku
multi sub handle(Int $n where * > 0) { say "Positive: $n" }
multi sub handle(Int $n where * <= 0) { die "Expected positive integer" }
multi sub handle(Str $s) { die "Expected integer, got string" }
```

### 15.5 Complex Validation with Smart Matching

Validate with `given`/`when` for complex validation logic:

```raku
given $input {
    when !*.defined { die "Input required" }
    when *.chars > 255 { die "Input too long" }
    when /^ \s* $/ { die "Input must not be blank" }
    default { process($input) }
}
```

### 15.6 Dangerous Operations

- Check collection bounds before indexing.
- Sanitize input used in file paths — validate against path traversal.
- Sanitize input used in system commands and regular expressions.
- Never use `EVAL` with unsanitized user input.
- Use the gradual type system: add types at boundaries where external
  data enters.

---

## 16. Project Structure

### 16.1 Recommended Layout

```
My-App/
    META6.json
    lib/
        My/
            App.rakumod
            App/
                UserService.rakumod
                Error.rakumod
    t/
        01-user-service.rakutest
    bin/
        my-app.raku
    resources/
    README.md
```

### 16.2 META6.json

```json
{
    "name": "My::App",
    "version": "0.1.0",
    "auth": "github:username",
    "depends": ["DBIish", "JSON::Fast"],
    "provides": {
        "My::App": "lib/My/App.rakumod",
        "My::App::UserService": "lib/My/App/UserService.rakumod"
    }
}
```

### 16.3 Dependency Management

- Use **Zef** for package installation.
- Declare dependencies in `META6.json`.
- Use `use v6.d;` to pin language version.

---

## 17. Tooling

| Purpose | Tool | Notes |
|---|---|---|
| Package manager | **Zef** | Install from ecosystem |
| Test runner | **prove6** or `zef test .` | TAP-compatible |
| Test framework | **Test** (built-in) | Core testing |
| Profiler | **Telemetry** | Built-in performance |
| REPL | **raku** (interactive) | Exploratory coding |
| Documentation | **Pod6** / `raku --doc` | Built-in doc extraction |

---

## 18. Build Tools

### 18.1 Zef (Package Manager & Build)

Raku's native package manager and build tool:

```bash
zef install MyModule  # Install from ecosystem
zef build .           # Build local module
zef test .            # Run tests
zef install --/test . # Install skipping tests
```

### 18.2 META6.json Configuration

Module metadata for building and dependency management:

```json
{
    "name": "MyApp",
    "version": "0.1.0",
    "description": "My Raku application",
    "author": "Author Name",
    "license": "MIT",
    "depends": [
        "Cro::HTTP:auth<zef:jnthn>:ver<0.3.3>",
        "Red:auth<zef:FCO>:ver<0.1.4>"
    ],
    "provides": {
        "MyApp": "lib/MyApp.rakumod"
    },
    "resources": [],
    "meta-version": "0"
}
```

### 18.3 Task Automation

Raku has no standard task-runner equivalent to Rake or Cargo's `xtask`.
Use a plain `Makefile` (most portable) or a small `bin/dev` script:

```makefile
.PHONY: test build release install
test:
	prove6 --lib t/
build:
	zef build .
install:
	zef install .
release:
	zef upload
```

For HTTP-service projects, `Cro::Tools` provides `cro stub`, `cro run`, and
`cro web` for scaffolding and running services — these are Cro-specific,
not a general task runner.

### 18.4 Build and Distribution

```bash
zef build  # Build the module
zef upload  # Upload to Raku ecosystem

# Or for local testing
raku -Ilib -e 'use MyApp; say "ok"'
```

### 18.5 Testing with prove6

Test harness for Raku:
```bash
prove6 --lib t/  # Run .rakutest files
```

### 18.6 Docker for Raku Applications

```dockerfile
FROM rakudo-star:latest
WORKDIR /app
COPY META6.json ./
RUN zef install --deps-only --/test .
COPY . .
CMD ["raku", "app.raku"]
```

---

## 19. SBOM Creation

### 19.1 What is an SBOM?

A Software Bill of Materials documents all Raku modules and dependencies. Essential for reproducibility, supply chain security, and vulnerability tracking.

### 19.2 META6.json and Dependency Management

Raku modules declare dependencies in `META6.json`:

```json
{
    "name": "MyApp",
    "version": "0.1.0",
    "depends": [
        "Cro::HTTP:auth<zef:jnthn>:ver<0.3.3>",
        "Red:auth<zef:FCO>:ver<0.1.4>",
        "JSON::Fast:auth<zef:timo>:ver<0.16>"
    ]
}
```

Install with Zef (locks versions):

```bash
zef install --deps-only .  # Install dependencies from META6.json
zef list --installed       # View installed modules
```

### 19.3 Version Pinning

Zef does **not** generate a lock file. To pin transitive dependencies for
reproducible installs, pin exact versions (and `auth`) in `META6.json`:

```json
"depends": [
    "Cro::HTTP:auth<zef:jnthn>:ver<0.3.3>",
    "JSON::Fast:auth<zef:timo>:ver<0.16>"
]
```

For full reproducibility, vendor dependencies (`zef fetch`) and commit
them, or build a known-good Docker image as the artefact.

### 19.4 SBOM Generation

**Generate from META6.json**:

```raku
use JSON::Fast;
my %meta = from-json(slurp 'META6.json');
my %sbom =
    name       => %meta<name>,
    version    => %meta<version>,
    components => %meta<depends>.map({ %( name => $_, type => 'module' ) }).List;
say to-json(%sbom);
```

`from-json` and `to-json` are both exported by `JSON::Fast`. The
component list must be materialised (`.List`) before JSON emission so it is
not lazily evaluated mid-serialisation.

Or export to CycloneDX format:

```bash
# Using a third-party tool if available
cyclonedx-raku --output sbom.json
```

### 19.5 Vulnerability Scanning

**Check Raku Advisory Database**:

- Monitor `https://github.com/Raku/ecosystem` for security advisories
- Subscribe to security mailing lists
- Use `zef info` to check module details

### 19.6 Integration into CI/CD

- Include `META6.json` in VCS (required)
- Pin exact versions (`:ver<x.y.z>`) and `auth` in `META6.json` — Zef has no lock file
- Generate SBOM on release
- Monitor Raku ecosystem for advisories
- Verify module licenses (documented in META6.json)
- Store SBOM with release artifacts

---

## 20. References

### Official Documentation

| Resource | URL |
|---|---|
| Raku Documentation | https://docs.raku.org |
| Raku Guide | https://raku.guide |
| Raku Modules | https://modules.raku.org |
| Raku Design Documents | https://design.raku.org |

### Books

| Book | Authors | Key Takeaways |
|---|---|---|
| *Think Raku* | Laurent Rosenfeld & Allen Downey (2020) | Practical introduction, grammars, concurrency, functional style. |
| *Perl 6 Deep Dive* | Andrew Shitov (2017) | Comprehensive language coverage — types, OO, concurrency, grammars. |
| *Perl 6 Fundamentals* | Moritz Lenz (2017) | Real-world projects using Raku features. |
| *Design Patterns* | Gamma et al. (1994) | Composition over inheritance; program to interfaces — roles in Raku. |
| *Clean Code* | Robert C. Martin (2008) | Small functions, meaningful names, SRP. |
