# Ruby Coding Style Guidelines

A comprehensive guide rooted in the community Ruby Style Guide, *The
Well-Grounded Rubyist*, *Practical Object-Oriented Design in Ruby* (Metz,
2012), and established software engineering literature — notably *Design
Patterns* (Gamma et al., 1994) and *Clean Code* (Martin, 2008).

---

## Table of Contents

1. [Philosophy](#1-philosophy)
2. [Code Layout & Formatting](#2-code-layout--formatting)
3. [Naming Conventions](#3-naming-conventions)
4. [Methods](#4-methods)
5. [Classes, Modules & Composition](#5-classes-modules--composition)
6. [Documentation & Comments](#6-documentation--comments)
7. [Error Handling](#7-error-handling)
8. [Requires & Gems](#8-requires--gems)
9. [Blocks, Procs & Lambdas](#9-blocks-procs--lambdas)
10. [Design Patterns in Ruby](#10-design-patterns-in-ruby)
11. [Testing](#11-testing)
12. [Database Access & ACID](#12-database-access--acid)
13. [Concurrency](#13-concurrency)
14. [Performance & Idiomatic Ruby](#14-performance--idiomatic-ruby)
15. [Defensive Programming & Input Validation](#15-defensive-programming--input-validation)
16. [Project Structure](#16-project-structure)
17. [Tooling](#17-tooling)
18. [Build Tools](#18-build-tools)
19. [SBOM Creation](#19-sbom-creation)
20. [References](#20-references)

---

## 1. Philosophy

### 1.1 Matz's Design Principle

> "Ruby is designed to make programmers happy."

Ruby optimises for **developer experience** — readability, expressiveness, and
the principle of least surprise.

### 1.2 Guiding Principles

| Principle | Source | Summary |
|---|---|---|
| **Readability over cleverness** | Ruby community, *Clean Code* Ch. 1 | Code should read like well-written prose. Optimise for the reader. |
| **Convention over configuration** | Ruby/Rails philosophy | Follow established conventions. Don't reinvent structure. |
| **Duck typing** | *The Well-Grounded Rubyist* | Depend on behaviour, not class identity. "If it quacks like a duck…" |
| **Composition over inheritance** | *POODR* Ch. 8, *Design Patterns* Ch. 1 | Prefer modules and delegation over deep inheritance hierarchies. |
| **Single Responsibility** | *Clean Code* Ch. 10, SOLID, *POODR* Ch. 4 | Every class and method should have one reason to change. |
| **YAGNI** | XP / *Clean Code* | Do not build for hypothetical future requirements. |
| **Least Surprise** | Matz, Ruby design | Methods should behave the way a reasonable Rubyist would expect. |

---

## 2. Code Layout & Formatting

### 2.1 RuboCop Is the Authority

All Ruby code **must** pass RuboCop with the project's agreed configuration.
RuboCop enforces the community Ruby Style Guide mechanically.

### 2.2 Indentation

- Use **2 spaces** per indentation level. Never use tabs.
- Continuation lines are indented one extra level.

### 2.3 Line Length

- **120 characters** per line, enforced by RuboCop (`Layout/LineLength`).
  Configure in `.rubocop.yml`:

```yaml
Layout/LineLength:
  Max: 120
```

### 2.4 Blank Lines (Ruby Style Guide, RuboCop)

RuboCop enforces these via `Layout/EmptyLines*` cops:

- **One blank line** between method definitions
  (`Layout/EmptyLineBetweenDefs`).
- **No blank line** after a class/module/method opening line
  (`Layout/EmptyLinesAroundClassBody`, `Layout/EmptyLinesAroundModuleBody`,
  `Layout/EmptyLinesAroundMethodBody`).
- **No blank line** before a class/module/method `end`
  (`Layout/EmptyLinesAroundClassBody`).
- **One blank line** between logical sections within a method.
- **Two blank lines** between class/module definitions in the same file.
- **No blank line** after a `begin` or before a `rescue`/`ensure`/`end`.
- **One blank line** between require groups (stdlib, gems, project).
- **No trailing blank lines** at the end of a file
  (`Layout/TrailingBlankLines`).

### 2.5 Whitespace

```ruby
# Yes
sum = a + b
array = [1, 2, 3]
hash = { key: value, other: data }

# No
sum=a+b
array = [1,2,3]
hash = {key: value,other: data}
```

### 2.6 Trailing Commas

Use trailing commas in multi-line arrays, hashes, and argument lists:

```ruby
FRUITS = [
  "apple",
  "banana",
  "cherry",
].freeze
```

### 2.7 String Literals

- Use **double quotes** for strings that contain interpolation or escapes.
- Use **single quotes** for plain string literals (no interpolation).
- Use **heredocs** for multi-line strings:

```ruby
message = <<~MSG
  Hello, #{name}!
  Welcome to the system.
MSG
```

### 2.8 The `frozen_string_literal` Pragma

Add the magic comment to every file to make string literals immutable by
default (preparing for Ruby's eventual default):

```ruby
# frozen_string_literal: true
```

---

## 3. Naming Conventions

| Entity | Convention | Example |
|---|---|---|
| Class | `PascalCase` | `UserService`, `HttpClient` |
| Module | `PascalCase` | `Serializable`, `Authentication` |
| Method | `snake_case` | `calculate_total` |
| Variable | `snake_case` | `total_count` |
| Instance variable | `@snake_case` | `@current_user` |
| Class variable | `@@snake_case` | `@@instance_count` (avoid — prefer class-level instance variables) |
| Constant | `UPPER_SNAKE_CASE` | `MAX_RETRIES`, `DEFAULT_TIMEOUT` |
| Predicate method | Ends with `?` | `empty?`, `valid?`, `admin?` |
| Dangerous method | Ends with `!` | `save!`, `sort!`, `delete!` |
| Setter method | Ends with `=` | `name=`, `status=` |
| File / require path | `snake_case` | `user_service.rb` |
| Gem | `kebab-case` or `snake_case` | `my-gem`, `my_gem` |

### 3.1 Naming Guidance (*Clean Code* Ch. 2)

- **Use intention-revealing names.** `elapsed_days` beats `d`.
- **Predicate methods return booleans** and read as questions: `user.active?`.
- **Bang methods (`!`)** indicate a "dangerous" variant — typically mutating
  in-place, raising on failure, or having irreversible side effects. Only add
  `!` when a non-bang counterpart exists.
- **No Hungarian notation or type prefixes.** Ruby is dynamically typed; the
  name should describe the role, not the type.
- **Acronyms follow `PascalCase` rules.** `HttpClient`, `JsonParser` — not
  `HTTPClient`, `JSONParser`.

---

## 4. Methods

### 4.1 Size and Focus (*Clean Code* Ch. 3, *POODR* Ch. 4)

- Methods should be **small** — ideally under 10 lines (Sandi Metz's rules).
- Each method does **one thing** at **one level of abstraction**.
- Limit classes to **100 lines** and methods to **5 lines** as aspirational
  targets (Sandi Metz's rules for exercising design discipline).

### 4.2 Parameters

- **Fewer parameters are better.** Beyond two positional parameters, use
  keyword arguments:

```ruby
# Yes — keyword arguments for clarity
def connect(host:, port:, timeout: 30, retries: 3)
  # ...
end

# No — positional arguments are opaque at the call site
def connect(host, port, timeout = 30, retries = 3)
  # ...
end
```

- **No boolean (flag) arguments.** Split into two methods.

### 4.3 Return Values

- Ruby methods **implicitly return** the last evaluated expression. Use
  explicit `return` only for **early exits** (guard clauses).
- **Be consistent** — a method should always return the same type. Don't
  return `nil` as a sentinel mixed with real values.
- Use `nil` to indicate absence; raise for errors.

### 4.4 Method Visibility

- Default to `private`. Only expose what clients need.
- Group visibility declarations at the bottom of the class:

```ruby
class UserService
  def find(id)
    # ...
  end

  def create(attrs)
    # ...
  end

  private

  def validate(attrs)
    # ...
  end

  def notify(user)
    # ...
  end
end
```

### 4.5 Avoid Lambdas for Named Logic

Use `Proc` or lambdas sparingly. Prefer **named methods** or **blocks** for
clarity:

```ruby
# Bad — anonymous, untestable
transform = ->(x) { x.strip.downcase.tr("-", "_") }

# Good — named, testable, shows up in stack traces
def normalise_key(value)
  value.strip.downcase.tr("-", "_")
end
```

Lambdas are acceptable as **strategy objects** passed to methods, but prefer a
dedicated class or module when the logic is non-trivial.

---

## 5. Classes, Modules & Composition

### 5.1 Class Design (*POODR*, *Clean Code*)

- Classes should be **small** — measured in responsibilities, not lines.
- Aim for **high cohesion**: every method should relate to the class's single
  purpose.
- Follow the **Single Responsibility Principle**: a class has one reason to
  change.

### 5.2 Modules and Mixins

Modules serve two purposes in Ruby: **namespacing** and **sharing behaviour**
(mixins).

```ruby
# Namespace
module Payments
  class Gateway
    # ...
  end
end

# Mixin — shared behaviour
module Loggable
  def log(message)
    logger.info("[#{self.class.name}] #{message}")
  end
end

class OrderService
  include Loggable

  def process(order)
    log("Processing order #{order.id}")
    # ...
  end
end
```

- Prefer **composition** (has-a via dependency injection) over **mixins**
  (includes) when the shared behaviour introduces state.
- Keep mixins **stateless** — they should add methods, not instance variables.
- Use `include` for instance methods, `extend` for class methods.

### 5.3 Inheritance vs. Composition (*POODR* Ch. 6–8)

> "Favour composition over inheritance."

- Use inheritance for **is-a** relationships and only when you control the
  base class.
- Use **composition** (delegation, dependency injection) for everything else.
- Prefer **duck typing** over `is_a?` checks. Depend on what an object
  *does*, not what it *is*.

```ruby
# Composition via dependency injection
class Archiver
  def initialize(compressor:)
    @compressor = compressor
  end

  def archive(payload)
    @compressor.compress(payload)
  end
end
```

### 5.4 Struct and Data (Ruby 3.2+)

Use `Struct` or `Data` for simple value objects:

```ruby
# Mutable value object
Coordinate = Struct.new(:latitude, :longitude, keyword_init: true)

# Immutable value object (Ruby 3.2+)
Coordinate = Data.define(:latitude, :longitude)

point = Coordinate.new(latitude: 48.8566, longitude: 2.3522)
```

- Use `Data.define` for immutable value objects when on Ruby 3.2+.
- Freeze structs when mutability is not needed.

### 5.5 SOLID in Ruby

| Principle | Ruby Application |
|---|---|
| **S** — Single Responsibility | One class per concern; extract service objects, query objects. |
| **O** — Open/Closed | Extend via modules, decorators, or subclassing — not by editing existing code. |
| **L** — Liskov Substitution | Subtypes and duck types must honour the contract of the type they replace. |
| **I** — Interface Segregation | Keep duck-type contracts small; don't force clients to implement methods they don't use. |
| **D** — Dependency Inversion | Inject dependencies through constructors; depend on duck types, not concretions. |

### 5.6 Typed Data Passing

Pass **typed objects** between methods and modules — not raw `Hash` instances.
Loosely-typed hashes obscure contracts, resist refactoring, and make callers
guess which keys exist.

- **Use `Struct`, `Data.define` (Ruby 3.2+), or custom classes** for data that
  crosses class or module boundaries.
- **At system boundaries** (API responses, config files, CLI arguments), parse
  into typed objects immediately — never pass raw hashes deeper into the
  application.
- **Return meaningful objects**, not arrays of loosely related values.
- Use `Struct.new` for simple mutable value objects and `Data.define` for
  immutable ones.

```ruby
# Bad — raw hash travels through multiple layers
def fetch_weather(city)
  response = HTTP.get("https://api.weather.example/#{city}")
  JSON.parse(response.body) # returns a Hash
end

forecast = fetch_weather("London")
logger.info(forecast["temperature"]) # caller must know the key name

# Good — parse into a typed object at the boundary
WeatherReport = Data.define(:city, :temperature, :humidity, :conditions)

def fetch_weather(city)
  raw = JSON.parse(HTTP.get("https://api.weather.example/#{city}").body)
  WeatherReport.new(
    city: raw.fetch("city"),
    temperature: raw.fetch("temperature"),
    humidity: raw.fetch("humidity"),
    conditions: raw.fetch("conditions"),
  )
end

report = fetch_weather("London")
logger.info(report.temperature) # clear contract, IDE-navigable
```

---

## 6. Documentation & Comments

### 6.1 YARD Doc Comments

Use **YARD** format for documenting public classes and methods:

```ruby
# Retrieve a user by their primary key.
#
# Looks up the user in the cache first, then falls back to the database.
#
# @param user_id [Integer] the unique identifier for the user
# @param include_deleted [Boolean] if true, soft-deleted users are returned
# @return [User] the matching user object
# @raise [UserNotFoundError] if no user matches the given id
def fetch_user(user_id, include_deleted: false)
  # ...
end
```

- **Start with a summary sentence** in imperative mood.
- Document all **parameters** with `@param`, **return values** with `@return`,
  and **exceptions** with `@raise`.
- Use `@example` for non-obvious usage:

```ruby
# Calculate compound interest.
#
# @param principal [Float] the initial amount
# @param rate [Float] annual interest rate (e.g. 0.05 for 5%)
# @param years [Integer] number of years
# @return [Float] the final amount
# @example
#   compound_interest(1000, 0.05, 10) #=> 1628.894626777442
def compound_interest(principal, rate, years)
  principal * (1 + rate)**years
end
```

### 6.2 Class-Level Documentation

```ruby
# Manages user authentication and session lifecycle.
#
# Uses JWT tokens for stateless authentication. Supports both
# symmetric (HS256) and asymmetric (RS256) signing algorithms.
#
# @example
#   auth = AuthService.new(secret: ENV["JWT_SECRET"])
#   token = auth.issue_token(user)
class AuthService
  # ...
end
```

### 6.3 Comments (*Clean Code* Ch. 4)

- Comments explain **why**, not what.
- Never commit commented-out code.
- Delete redundant comments that restate the code.
- Use `# TODO:` and `# FIXME:` sparingly — track real issues in the issue
  tracker.

---

## 7. Error Handling

### 7.1 Principles

- **Raise exceptions for exceptional conditions.** Do not use return codes.
- **Rescue specific exceptions.** Never bare-`rescue` (`rescue => e` catches
  only `StandardError`, which is correct; `rescue Exception` catches *everything*
  including `SystemExit` and `SignalException` — almost never appropriate).
- **Fail fast.** Validate at the boundary, then trust downstream.

### 7.2 Exception Hierarchy

Define domain exceptions inheriting from `StandardError`:

```ruby
module MyApp
  class Error < StandardError; end
  class NotFoundError < Error; end
  class PermissionDeniedError < Error; end
  class ValidationError < Error; end
end
```

- Always inherit from `StandardError`, never from `Exception`.
- Group exceptions under a module-level base class.

### 7.3 `ensure` for Cleanup (Context Manager Equivalent)

Ruby's `ensure` block is the equivalent of Python's `with` / Go's `defer`.
**Always** use `ensure` (or a block-based API) for resource cleanup:

```ruby
def read_file(path)
  file = File.open(path)
  file.read
ensure
  file&.close
end
```

Prefer the **block form** when available — it handles cleanup automatically:

```ruby
# Yes — block form, cleanup is automatic
File.open("data.csv") do |f|
  f.each_line { |line| process(line) }
end

# No — manual open/close, leak-prone
f = File.open("data.csv")
f.each_line { |line| process(line) }
f.close
```

#### Writing Custom Block-Based APIs

Provide block-accepting methods for resources that need cleanup:

```ruby
class DatabasePool
  # Yield a connection from the pool, returning it on completion.
  #
  # @yield [Connection] an active database connection
  # @raise [ConnectionError] if no connection is available
  def with_connection
    conn = checkout
    yield conn
  ensure
    checkin(conn) if conn
  end
end

# Usage
pool.with_connection do |conn|
  conn.execute("SELECT 1")
end
```

### 7.4 `retry` with Limits

```ruby
def fetch_with_retry(url, max_attempts: 3)
  attempts = 0
  begin
    attempts += 1
    HTTP.get(url)
  rescue HTTP::TimeoutError => e
    raise if attempts >= max_attempts

    sleep(2**attempts)
    retry
  end
end
```

### 7.5 Logging Over `puts`

- Use a logger (`Logger`, `Semantic Logger`, `Rails.logger`), never `puts` or
  `p`, for operational output.
- Log at the appropriate level: `debug`, `info`, `warn`, `error`, `fatal`.

---

## 8. Requires & Gems

### 8.1 Require Ordering

Group `require` statements separated by blank lines:

1. **Standard library** (`json`, `net/http`, `fileutils`)
2. **Third-party gems** (`httparty`, `dry-struct`)
3. **Internal / project** (`require_relative`)

```ruby
require "json"
require "net/http"

require "httparty"
require "dry-struct"

require_relative "config"
require_relative "services/auth"
```

### 8.2 One Gem per Concern — No Redundant Dependencies

When multiple gems accomplish the same task, **pick one and use it
consistently**:

| Overlapping gems | Pick one |
|---|---|
| `httparty` / `faraday` / `typhoeus` | One HTTP client per project |
| `rspec` / `minitest` | One test framework |
| `haml` / `slim` / `erb` | One template engine |
| `nokogiri` / `oga` for XML/HTML parsing | One parser |
| `puma` / `unicorn` / `falcon` | One app server |
| `dotenv` / `figaro` for env config | One config loader |
| `sidekiq` / `resque` / `good_job` | One background job processor |

### 8.3 Gemfile Discipline

- **Pin versions** in the `Gemfile` with pessimistic constraints (`~>`).
- Separate `:development` and `:test` groups from production dependencies.
- Run `bundle audit` regularly for vulnerability scanning.
- Commit `Gemfile.lock` to version control.

---

## 9. Blocks, Procs & Lambdas

### 9.1 Blocks

Blocks are Ruby's primary tool for callbacks and resource management. Prefer
**blocks** for single-use logic:

```ruby
# Iteration
users.each { |user| send_welcome(user) }

# Multi-line blocks use do...end
users.each do |user|
  validate(user)
  send_welcome(user)
end
```

**Convention:** use `{ }` for single-line blocks and `do...end` for multi-line.

### 9.2 Procs vs. Lambdas

| Feature | `Proc.new` / `proc` | `lambda` / `->` |
|---|---|---|
| Arity checking | Lenient (ignores extra args) | Strict (raises `ArgumentError`) |
| `return` behaviour | Returns from **enclosing method** | Returns from **the proc only** |
| Recommended for | Rarely — surprises with `return` | Strategy objects, callbacks |

**Prefer lambdas** over `Proc.new` when you need a callable object. Their
strict arity and local `return` semantics are less surprising:

```ruby
# Yes — lambda
validator = ->(value) { value.is_a?(String) && !value.empty? }

# No — proc (return would exit the enclosing method)
validator = Proc.new { |value| value.is_a?(String) && !value.empty? }
```

### 9.3 Method Objects

Use `method(:name)` to pass a named method as a callable — this is preferable
to lambdas for non-trivial logic:

```ruby
def normalise(value)
  value.strip.downcase
end

# Pass the named method as a block
names.map(&method(:normalise))
```

### 9.4 `Symbol#to_proc`

Use the `&:method_name` shorthand for simple method calls:

```ruby
# Yes
names.map(&:downcase)
users.select(&:active?)

# No — unnecessary lambda
names.map { |n| n.downcase }
```

---

## 10. Design Patterns in Ruby

The GoF patterns remain relevant, but Ruby's dynamic typing, blocks, modules,
and open classes simplify many implementations.

### 10.1 Creational Patterns

#### Factory Method

Use class methods or standalone factory functions:

```ruby
class Notification
  # Create a notification appropriate to the event's severity.
  #
  # @param event [Event] the domain event
  # @return [Notification] an urgent or standard notification
  def self.from_event(event)
    case event.severity
    when :critical then UrgentNotification.new(event.message)
    else StandardNotification.new(event.message)
    end
  end
end
```

#### Builder

Use keyword arguments or a dedicated builder with a fluent interface:

```ruby
class QueryBuilder
  def initialize(table)
    @table = table
    @conditions = []
    @limit = nil
  end

  # Add a WHERE condition.
  #
  # @param condition [String] a SQL condition
  # @return [QueryBuilder] self for chaining
  def where(condition)
    @conditions << condition
    self
  end

  # Set the LIMIT clause.
  #
  # @param n [Integer] maximum rows
  # @return [QueryBuilder] self for chaining
  def limit(n)
    @limit = n
    self
  end

  # Render the final SQL string.
  #
  # @return [String] a SQL SELECT statement
  def build
    sql = "SELECT * FROM #{@table}"
    sql += " WHERE #{@conditions.join(' AND ')}" if @conditions.any?
    sql += " LIMIT #{@limit}" if @limit
    sql
  end
end
```

#### Singleton

Use Ruby's built-in `Singleton` module:

```ruby
require "singleton"

class Registry
  include Singleton

  def initialize
    @entries = {}
  end

  # Register a service under the given name.
  #
  # @param name [Symbol] the service name
  # @param service [Object] the service instance
  def register(name, service)
    @entries[name] = service
  end
end
```

### 10.2 Structural Patterns

#### Decorator

Use `SimpleDelegator` or manual delegation:

```ruby
require "delegate"

class LoggingClient < SimpleDelegator
  # Wrap an HTTP client with request logging.
  #
  # @param client [HttpClient] the client to decorate
  # @param logger [Logger] the logger instance
  def initialize(client, logger:)
    super(client)
    @logger = logger
  end

  def get(url)
    @logger.info("GET #{url}")
    super
  end
end
```

#### Adapter

Wrap an incompatible interface:

```ruby
class LegacyPrinterAdapter
  # @param legacy [LegacyPrinter] the legacy printer to wrap
  def initialize(legacy)
    @legacy = legacy
  end

  # Print text via the standard interface.
  #
  # @param text [String] the text to print
  def print(text)
    @legacy.print_old(text)
  end
end
```

### 10.3 Behavioural Patterns

#### Strategy

Pass a callable or an object responding to a known message:

```ruby
class Archiver
  # @param compressor [#compress] any object responding to `compress`
  def initialize(compressor:)
    @compressor = compressor
  end

  # Archive the payload using the configured strategy.
  #
  # @param payload [String] raw data to archive
  # @return [String] compressed data
  def archive(payload)
    @compressor.compress(payload)
  end
end
```

#### Observer

Use Ruby's `Observable` module or a custom event emitter:

```ruby
class EventEmitter
  def initialize
    @listeners = Hash.new { |h, k| h[k] = [] }
  end

  # Register a callback for the given event.
  #
  # @param event [Symbol] the event name
  # @yield [Object] the event payload
  def on(event, &callback)
    @listeners[event] << callback
  end

  # Fire an event.
  #
  # @param event [Symbol] the event name
  # @param payload [Object] data passed to each listener
  def emit(event, payload = nil)
    @listeners[event].each { |cb| cb.call(payload) }
  end
end
```

#### Template Method

Use inheritance with abstract-like methods:

```ruby
class DataPipeline
  # Execute the full extract-transform-load cycle.
  #
  # @param source [String] path or URI to the data source
  def run(source)
    raw = extract(source)
    data = transform(raw)
    load(data)
  end

  private

  def extract(source)
    raise NotImplementedError
  end

  def transform(raw)
    raise NotImplementedError
  end

  def load(data)
    raise NotImplementedError
  end
end
```

---

## 11. Testing

### 11.1 Principles

- Use **RSpec** or **Minitest** — pick one per project and stay consistent.
- **Test behaviour, not implementation.** Tests must survive refactors.
- Follow the **testing pyramid**: unit > integration > e2e.
- Tests are **first-class code** — same naming and quality standards apply.

### 11.2 RSpec Naming

Describe the **class or method** and the **scenario**:

```ruby
RSpec.describe Account do
  describe "#withdraw" do
    context "when funds are insufficient" do
      it "raises an InsufficientFundsError" do
        account = Account.new(balance: 100)
        expect { account.withdraw(200) }.to raise_error(InsufficientFundsError)
      end
    end

    context "when funds are sufficient" do
      it "reduces the balance by the withdrawal amount" do
        account = Account.new(balance: 100)
        account.withdraw(30)
        expect(account.balance).to eq(70)
      end
    end
  end
end
```

### 11.3 Structure (Arrange-Act-Assert)

```ruby
it "applies discount to the cart total" do
  # Arrange
  cart = ShoppingCart.new(items: [Item.new("Book", price: 20.00)])
  discount = PercentageDiscount.new(10)

  # Act
  cart.apply_discount(discount)

  # Assert
  expect(cart.total).to eq(18.00)
end
```

### 11.4 Factories Over Fixtures

- Use **FactoryBot** for building test data — not fixtures, not raw
  constructors scattered across tests.
- Keep factories minimal. Override attributes in individual tests.

### 11.5 Mocking

- Prefer **dependency injection** over stubbing. If you must stub, stub at
  the **boundary** (HTTP, database, clock).
- Use `instance_double` for type-checked test doubles.
- Never mock the object under test.

---

## 12. Database Access & ACID

When interacting with SQL databases, every query and transaction must respect
the **ACID** properties — **Atomicity**, **Consistency**, **Isolation**, and
**Durability**.

### 12.1 ACID at a Glance

| Property | Guarantee | Violation Example |
|---|---|---|
| **Atomicity** | A transaction either completes entirely or has no effect. | A transfer debits one account but crashes before crediting the other. |
| **Consistency** | A transaction moves the database from one valid state to another. | An insert succeeds despite violating a foreign key constraint. |
| **Isolation** | Concurrent transactions do not interfere with each other. | Two threads read the same balance, both withdraw, and the final balance is wrong. |
| **Durability** | Once committed, data survives crashes and power loss. | A commit returns successfully but the data is lost after a restart. |

### 12.2 Always Use Explicit Transactions

```ruby
ActiveRecord::Base.transaction do
  source.update!(balance: source.balance - amount)
  target.update!(balance: target.balance + amount)
end
```

- ActiveRecord's `transaction` block rolls back automatically if any
  exception is raised.
- Use `update!` (bang methods) inside transactions so validation failures
  trigger a rollback.
- Keep transactions **short** — hold locks for the minimum time necessary.

### 12.3 Choose the Correct Isolation Level

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Use When |
|---|---|---|---|---|
| READ UNCOMMITTED | Possible | Possible | Possible | Almost never |
| READ COMMITTED | No | Possible | Possible | Default for most OLTP workloads |
| REPEATABLE READ | No | No | Possible | Reports needing a stable snapshot |
| SERIALIZABLE | No | No | No | Financial transactions, inventory |

```ruby
ActiveRecord::Base.transaction(isolation: :serializable) do
  # critical operations
end
```

### 12.4 Use Parameterised Queries

```ruby
# Yes — parameterised
User.where("email = ?", email)
User.where(email: email)

# No — string interpolation (SQL injection)
User.where("email = '#{email}'")
```

### 12.5 Connection Lifecycle

- Use the framework's **connection pool** (ActiveRecord's `pool` setting).
- Never hold a connection across HTTP request boundaries.
- Use `ActiveRecord::Base.connection_pool.with_connection` for background
  threads.

### 12.6 Locking

Use `SELECT ... FOR UPDATE` via ActiveRecord:

```ruby
ActiveRecord::Base.transaction do
  account = Account.lock.find(account_id)
  account.update!(balance: account.balance - amount)
end
```

### 12.7 SQL Injection Protection

SQL injection remains one of the most dangerous and common vulnerabilities.
**Never** build SQL queries with string interpolation or concatenation.

- **Always use parameterised queries** or ActiveRecord's built-in escaping.
- Use ActiveRecord hash conditions or placeholder syntax.
- Validate and constrain input before it reaches the database layer.

```ruby
# Bad — string interpolation (SQL injection vector)
User.where("name = '#{params[:name]}'")
User.where("role = '#{role}' AND active = #{active}")

# Good — hash conditions
User.where(name: params[:name])

# Good — positional placeholders
User.where("role = ? AND active = ?", role, active)

# Good — named placeholders
User.where("role = :role AND active = :active", role: role, active: active)
```

When using raw SQL through `execute` or `find_by_sql`, always use bind
parameters:

```ruby
# Bad — string interpolation
ActiveRecord::Base.connection.execute(
  "DELETE FROM sessions WHERE token = '#{token}'"
)

# Good — find_by_sql with positional bindings
Session.find_by_sql(["SELECT * FROM sessions WHERE token = ?", token])

# Good — exec_query with type-aware binds (avoids any quoting)
ActiveRecord::Base.connection.exec_query(
  "DELETE FROM sessions WHERE token = $1",
  "delete sessions",
  [ActiveRecord::Relation::QueryAttribute.new(
    "token", token, ActiveRecord::Type::String.new
  )],
)
```

---

## 13. Concurrency

### 13.1 The GIL (GVL) Reality

CRuby's Global VM Lock (GVL) means **only one thread runs Ruby code** at a
time. Threads are still useful for I/O-bound work (HTTP, database), but not
for CPU-bound parallelism.

| Task Type | Recommended Approach |
|---|---|
| I/O-bound (HTTP, DB, files) | Threads, Fibers (Ruby 3+), or async gems |
| CPU-bound (compute) | `Ractor` (Ruby 3+) or spawn separate processes |
| Background jobs | Sidekiq, GoodJob, SolidQueue |

### 13.2 Thread Safety

- Avoid shared mutable state. If unavoidable, protect it with a `Mutex`.
- Prefer **immutable objects** (frozen strings, `Data.define`).
- Use thread-safe data structures (`Concurrent::Hash` from `concurrent-ruby`).

### 13.3 Fibers and Ractors (Ruby 3+)

- **Fibers** provide cooperative concurrency — useful with async I/O gems
  (`async`, `falcon`).
- **Ractors** provide true parallelism by isolating state between actors.
  They are experimental and have strict sharing rules.

---

## 14. Performance & Idiomatic Ruby

### 14.1 Enumerable Methods

Prefer `Enumerable` methods over manual loops:

```ruby
# Yes
active_users = users.select(&:active?)
names = users.map(&:name)
total = items.sum(&:price)

# No
active_users = []
users.each { |u| active_users << u if u.active? }
```

### 14.2 Guard Clauses

Return early instead of nesting:

```ruby
def process_order(order)
  raise OrderCancelledError if order.cancelled?
  return Receipt.empty if order.items.empty?

  build_receipt(order)
end
```

### 14.3 Freeze Constants

Freeze mutable constants to prevent accidental mutation:

```ruby
VALID_STATUSES = %w[active inactive suspended].freeze
DEFAULT_CONFIG = { timeout: 30, retries: 3 }.freeze
```

### 14.4 Copy Semantics

Ruby variables hold **references**. Assignment does not copy objects:

```ruby
a = [1, 2, 3]
b = a
b << 4
a # => [1, 2, 3, 4] — same object!
```

| Operation | Outer | Nested | Use when |
|---|---|---|---|
| `b = a` | Shared | Shared | You want an alias |
| `a.dup` | New | Shared | Flat structures, shallow copy |
| `a.clone` | New (preserves frozen state) | Shared | Need frozen state preserved |
| `Marshal.load(Marshal.dump(a))` | New | New | Deep copy (slow, no procs/IO) |

- Use `.dup` for defensive copies at method boundaries.
- Use `.freeze` to make objects immutable and prevent accidental mutation.
- Use `deep_dup` (ActiveSupport) for nested structures in Rails projects.

### 14.5 Avoid `method_missing`

`method_missing` is powerful but invisible to tools, breaks `respond_to?`,
and makes debugging painful. Prefer explicit method definitions or
`define_method` with `respond_to_missing?` if dynamic dispatch is truly
necessary.

### 14.6 String Interpolation Over Concatenation

```ruby
# Yes
greeting = "Hello, #{name}!"

# No
greeting = "Hello, " + name + "!"
```

---

## 15. Defensive Programming & Input Validation

### 15.1 Validate at Boundaries

All external input — controller parameters, API payloads, file contents, CLI
arguments, environment variables — must be validated at the entry point before
it flows deeper into the application.

- Use **Rails Strong Parameters** for web input filtering.
- Use `raise ArgumentError` for invalid input at method boundaries.

```ruby
def transfer(from_id:, to_id:, amount:)
  raise ArgumentError, "amount must be positive" unless amount.positive?
  raise ArgumentError, "cannot transfer to self" if from_id == to_id
  # ...
end
```

### 15.2 Type Conversion

Use strict conversion methods that raise on invalid input instead of silently
returning zero or nil:

```ruby
# Bad — silently returns 0
"abc".to_i   # => 0

# Good — raises ArgumentError
Integer("abc")   # => ArgumentError
Float("abc")     # => ArgumentError
```

Use `Kernel#Integer()` and `Kernel#Float()` for strict parsing. Reserve
`to_i` / `to_f` for cases where silent coercion is intentional.

### 15.3 String and Numeric Validation

- **Validate string lengths** — check `.length` or `.size` against limits
  before processing or storing.
- **Validate numeric ranges** — check against bounds before arithmetic or
  database writes.

```ruby
def update_username(user, new_name)
  raise ArgumentError, "username too long (max 50)" if new_name.length > 50
  raise ArgumentError, "username cannot be blank" if new_name.strip.empty?
  user.update!(username: new_name)
end

def set_quantity(item, qty)
  raise ArgumentError, "quantity must be 1..9999" unless (1..9999).cover?(qty)
  item.update!(quantity: qty)
end
```

### 15.4 Collection Safety

- Check collection bounds before indexing.
- Use `.fetch` with a default or block for hash access — it raises `KeyError`
  on missing keys instead of returning `nil` silently.

```ruby
# Bad — silent nil on missing key
config = { timeout: 30 }
config[:retries]  # => nil (bug hides)

# Good — explicit failure
config.fetch(:retries)              # => KeyError
config.fetch(:retries, 3)           # => 3 (explicit default)
config.fetch(:retries) { compute }  # => computed default
```

### 15.5 Path and Shell Safety

- Sanitize input used in file paths with `File.expand_path` and `Pathname`.
  Reject paths that escape the expected directory.
- Sanitize input used in shell commands with `Shellwords.escape`. Prefer
  direct APIs over shelling out whenever possible.

```ruby
# Bad — path traversal
File.read("uploads/#{params[:filename]}")

# Good — constrain to a directory
safe_path = Pathname.new("uploads").join(params[:filename]).cleanpath
raise ArgumentError, "invalid path" unless safe_path.to_s.start_with?("uploads/")
File.read(safe_path)
```

### 15.6 Dangerous Methods

- **Never** use `eval`, `send`, `public_send`, or `instance_eval` with user
  input. These methods execute arbitrary code and are a direct path to remote
  code execution vulnerabilities.
- Use `frozen_string_literal: true` to prevent accidental string mutation.

---

## 16. Project Structure

### 16.1 Recommended Layout (Gem)

```
my_gem/
    my_gem.gemspec
    Gemfile
    Gemfile.lock
    Rakefile
    README.md
    LICENSE
    lib/
        my_gem.rb            # entry point, requires submodules
        my_gem/
            version.rb
            client.rb
            errors.rb
    spec/                    # or test/ for Minitest
        spec_helper.rb
        my_gem/
            client_spec.rb
    sig/                     # RBS type signatures (Ruby 3+)
        my_gem.rbs
```

### 16.2 Recommended Layout (Rails Application)

Follow the Rails conventions — `app/models`, `app/controllers`,
`app/services`, `app/jobs`, etc. Extract domain logic into service objects
and keep controllers thin.

### 16.3 Dependency Management

- Use **Bundler** for all dependency management.
- Pin versions with pessimistic constraints: `gem "rails", "~> 7.1"`.
- Commit `Gemfile.lock`.
- Run `bundle audit` in CI.

---

## 17. Tooling

### 17.1 Recommended Tool Chain

| Purpose | Tool | Notes |
|---|---|---|
| Linter / Formatter | **RuboCop** | Community style guide enforcement |
| Test framework | **RSpec** or **Minitest** | Pick one per project |
| Factory library | **FactoryBot** | Build test data |
| Coverage | **SimpleCov** | Line and branch coverage |
| Security scanner | **Brakeman** (Rails), **bundler-audit** | Vulnerability detection |
| Type checking | **Sorbet** or **RBS + Steep** | Gradual typing |
| Documentation | **YARD** | Generate HTML docs from annotations |
| Pre-commit | **Overcommit** or **Lefthook** | Run cops and tests on commit |

### 17.2 CI Checks

```bash
bundle exec rubocop                # lint + format
bundle exec rspec                  # tests
bundle exec bundler-audit check    # dependency vulnerabilities
bundle exec brakeman -q            # security (Rails)
```

---

## 18. Build Tools

### 18.1 Rake (Task Automation)

Rake is Ruby's standard task automation tool (like Make):

```ruby
# Rakefile
task :build do
  sh 'bundle install'
  sh 'gem build my_gem.gemspec'
end

task :test do
  sh 'bundle exec rspec'
end

task :release => :build do
  sh 'gem push pkg/my_gem-*.gem'
end
```

Run tasks:
```bash
rake build
rake test
rake -T  # List all tasks
```

### 18.2 Bundler for Dependency Management

Bundler locks dependencies for reproducible builds:

```bash
bundle install  # Install from Gemfile.lock
bundle update  # Update dependencies
bundle exec ruby script.rb  # Run in Bundler environment
```

Gemfile:
```ruby
source 'https://rubygems.org'
gem 'rails', '~> 7.1'
gem 'pg', '~> 1.5'
gem 'puma', '~> 6.0'

group :development do
  gem 'rspec-rails'
  gem 'pry-rails'
end
```

### 18.3 Gemspec for Packaging

Define gem metadata in `.gemspec`:
```ruby
Gem::Specification.new do |spec|
  spec.name = 'my_gem'
  spec.version = '1.0.0'
  spec.authors = ['Author']
  spec.email = ['author@example.com']
  spec.summary = 'Brief description'
  spec.files = Dir['lib/**/*.rb']
  spec.add_dependency 'rails', '~> 7.1'
  spec.add_development_dependency 'rspec', '~> 3.0'
end
```

Build and publish:
```bash
gem build my_gem.gemspec
gem push pkg/my_gem-1.0.0.gem
```

### 18.4 Rails Asset Pipeline

Rails 7+ defaults to **Propshaft + import maps** (Sprockets is now optional/legacy).
For modern JS bundling, use **jsbundling-rails** (esbuild/rollup/webpack) or Shakapacker:

```bash
bundle exec rails assets:precompile  # Compile assets
bundle exec rails assets:clobber  # Clean compiled assets
```

Add a JS bundler:
```bash
# jsbundling-rails (recommended for Rails 7+)
bundle add jsbundling-rails
bin/rails javascript:install:esbuild  # or :rollup, :webpack

# Or Shakapacker for webpack-only setups
bundle add shakapacker
bundle exec rails javascript:install:webpack
```

### 18.5 Docker for Ruby Applications

Dockerfile for Rails apps:
```dockerfile
FROM ruby:3.2-alpine
WORKDIR /app
COPY Gemfile Gemfile.lock ./
RUN bundle install --without development test
COPY . .
RUN bundle exec rails assets:precompile
CMD ["bundle", "exec", "rails", "server"]
```

### 18.6 Build Automation with GitHub Actions

```yaml
# .github/workflows/build.yml
name: Build & Test
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: ruby/setup-ruby@v1
      - run: bundle install
      - run: bundle exec rake test
```

---

## 19. SBOM Creation

### 19.1 What is an SBOM?

A Software Bill of Materials lists all gems (dependencies) and their versions. Critical for security audits, vulnerability tracking, and license compliance.

### 19.2 Bundler and Dependency Locking

- `Gemfile`: Declares direct dependencies
- `Gemfile.lock`: Records exact versions for reproducible builds (always include in VCS)

Generate dependency tree:
```bash
bundle info --path <gem>
bundle info all  # List all gems
```

### 19.3 SBOM Generation with CycloneDX

**Using `cyclonedx-ruby`**:
```bash
gem install cyclonedx-ruby
cyclonedx-ruby  # Generates sbom.json and sbom.xml
cyclonedx-ruby --output-file sbom.json
```

### 19.4 Vulnerability Scanning

**Using `bundler-audit`** (checks Ruby Advisory Database):
```bash
gem install bundler-audit
bundle audit check --update  # Check Gemfile.lock for vulns
bundle audit --json > audit-report.json
```

**GitHub Dependabot** (built-in for GitHub):
- Automatically scans `Gemfile.lock`
- Creates PRs for vulnerable updates
- Dashboard visibility on Security tab

### 19.5 License Compliance

**Using `license_finder`**:
```bash
gem install license_finder
license_finder report --format json > licenses.json
license_finder approve_license BSD
license_finder restrict_to MIT, Apache-2.0  # Whitelist licenses
```

### 19.6 Integration into CI/CD

- Lock `Gemfile.lock` in VCS (required for reproducibility)
- Run `bundle audit` on every commit/PR
- Generate SBOM with `cyclonedx-ruby` on release
- Check licenses with `license_finder`; fail if non-whitelisted
- Monitor Ruby Advisory Database for updates
- Store SBOM and audit reports as release artifacts

---

## 20. References

### Official & Community Documentation

| Resource | URL |
|---|---|
| Ruby Style Guide (community) | https://rubystyle.guide |
| RuboCop Documentation | https://docs.rubocop.org |
| Ruby Documentation | https://ruby-doc.org |
| YARD Documentation | https://yardoc.org |
| RBS (Type Signatures) | https://github.com/ruby/rbs |

### Books

| Book | Authors | Key Takeaways for This Guide |
|---|---|---|
| *Practical Object-Oriented Design in Ruby* | Sandi Metz (2012, 2018) | SRP, dependency injection, duck typing, composition over inheritance — the definitive Ruby design book. |
| *The Well-Grounded Rubyist* | David A. Black (2009, 2019) | Deep understanding of Ruby's object model, blocks, modules, and method lookup. |
| *Eloquent Ruby* | Russ Olsen (2011) | Writing idiomatic, readable Ruby — from naming to metaprogramming. |
| *Ruby Under a Microscope* | Pat Shaughnessy (2013) | Internals: how Ruby parses, compiles, and executes code. |
| *Design Patterns: Elements of Reusable Object-Oriented Software* | Gamma, Helm, Johnson, Vlissides (1994) | Favour composition over inheritance; program to interfaces. |
| *Clean Code: A Handbook of Agile Software Craftsmanship* | Robert C. Martin (2008) | Small methods, meaningful names, SRP — applies to any language. |
| *Refactoring: Ruby Edition* | Fields, Harvie, Fowler (2009) | Systematic code improvement techniques adapted for Ruby. |
