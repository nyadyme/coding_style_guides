# C# Coding Style Guidelines

A comprehensive guide rooted in the Microsoft C# Coding Conventions,
.NET Framework Design Guidelines, *C# in Depth* (Skeet, 2019), *Effective
C#* (Wagner, 2017), and established software engineering literature —
notably *Design Patterns* (Gamma et al., 1994) and *Clean Code*
(Martin, 2008).

---

## Table of Contents

1. [Philosophy](#1-philosophy)
2. [Code Layout & Formatting](#2-code-layout--formatting)
3. [Naming Conventions](#3-naming-conventions)
4. [Methods](#4-methods)
5. [Types & Composition](#5-types--composition)
6. [Documentation & Comments](#6-documentation--comments)
7. [Error Handling](#7-error-handling)
8. [Namespaces & Imports](#8-namespaces--imports)
9. [Design Patterns in C#](#9-design-patterns-in-c)
10. [Testing](#10-testing)
11. [Database Access & ACID](#11-database-access--acid)
12. [Concurrency & Async](#12-concurrency--async)
13. [Performance & Idiomatic C#](#13-performance--idiomatic-c)
14. [Defensive Programming & Input Validation](#14-defensive-programming--input-validation)
15. [Project Structure](#15-project-structure)
16. [Tooling](#16-tooling)
17. [Build Tools](#17-build-tools)
18. [SBOM Creation](#18-sbom-creation)
19. [References](#19-references)

---

## 1. Philosophy

### 1.1 C#'s Core Values

C# is a strongly-typed, multi-paradigm language designed for safety,
productivity, and performance. Modern C# embraces pattern matching,
records, nullable reference types, and async/await as core language
features.

### 1.2 Guiding Principles

| Principle | Source | Summary |
|---|---|---|
| **Type safety first** | C# design, *Effective C#* | Leverage the type system to prevent bugs at compile time. |
| **Nullable reference types** | C# 8.0+ | Enable `<Nullable>enable</Nullable>` — eliminate null reference exceptions. |
| **Immutability by default** | *Effective C#*, functional C# | Prefer `readonly`, records, and immutable collections. |
| **Composition over inheritance** | *Design Patterns* Ch. 1 | Prefer interfaces and dependency injection over deep hierarchies. |
| **Single Responsibility** | *Clean Code* Ch. 10, SOLID | Every class and method should have one reason to change. |
| **YAGNI** | XP / *Clean Code* | Do not build for hypothetical future requirements. |
| **Convention over configuration** | .NET ecosystem | Follow .NET conventions — they are well-established and widely understood. |

---

## 2. Code Layout & Formatting

### 2.1 Formatter

Use **dotnet format** (built-in) or configure via `.editorconfig`.

### 2.2 Indentation

- **4 spaces** per indentation level. Never tabs.

### 2.3 Line Length

- **120 characters** recommended maximum.

### 2.4 Blank Lines

- **One blank line** between method definitions.
- **One blank line** between property groups and method groups.
- **No blank line** after an opening brace or before a closing brace.
- **One blank line** between `using` directive groups (system, third-party,
  project).
- Use single blank lines within methods to separate logical sections.
- **No trailing blank lines** at the end of a file.

### 2.5 Braces

Allman style (opening brace on its own line) per Microsoft convention:

```csharp
if (condition)
{
    DoSomething();
}
else
{
    DoOther();
}

public void Process(List<Item> items)
{
    foreach (var item in items)
    {
        Handle(item);
    }
}
```

### 2.6 File Organisation

1. `using` directives
2. Namespace declaration
3. Type declaration(s) — one primary type per file

Within a type:

1. Constants and static readonly fields
2. Fields
3. Constructors
4. Properties
5. Public methods
6. Private methods
7. Nested types

### 2.7 File-Scoped Namespaces (C# 10+)

```csharp
namespace MyApp.Services;

public class UserService
{
    // ...
}
```

---

## 3. Naming Conventions

| Entity | Convention | Example |
|---|---|---|
| Namespace | `PascalCase.Separated` | `MyApp.Services` |
| Class / Struct / Record | `PascalCase` | `UserService`, `OrderItem` |
| Interface | `IPascalCase` | `IUserRepository` |
| Method | `PascalCase` | `CalculateTotal()` |
| Property | `PascalCase` | `FirstName` |
| Event | `PascalCase` | `OrderPlaced` |
| Enum type | `PascalCase` | `OrderStatus` |
| Enum member | `PascalCase` | `OrderStatus.Pending` |
| Public constant | `PascalCase` | `MaxRetries` |
| Local variable | `camelCase` | `totalCount` |
| Parameter | `camelCase` | `userId` |
| Private field | `_camelCase` | `_userRepository` |
| Static private field | `s_camelCase` | `s_instance` |
| Type parameter | `T` prefix | `TKey`, `TValue` |
| Boolean | Prefix `Is`, `Has`, `Can` | `IsValid`, `HasPermission` |
| Async method | `Async` suffix | `GetUserAsync()` |
| File | Matches primary type | `UserService.cs` |

### 3.1 Naming Guidance

- Meaningful, intention-revealing names. `elapsedSeconds` beats `s`.
- No Hungarian notation. No member prefixes other than `_` for private fields.
- Acronyms: two-letter acronyms are all caps (`IO`), three+ are PascalCase
  (`Xml`, `Json`).
- Interface names describe capabilities (`IDisposable`) or contracts
  (`IUserRepository`).

---

## 4. Methods

### 4.1 Size and Focus

- Small (under 20 lines), one thing, one level of abstraction.

### 4.2 Parameters

- Maximum 3–4 parameters. Group related parameters into a record or class.
- Use `in` for read-only by-reference passing of large structs.
- Use optional parameters or method overloads for defaults.

```csharp
public record ConnectionOptions(int Timeout = 30, int Retries = 3);

public void Connect(string host, int port, ConnectionOptions? options = null)
{
    var opts = options ?? new ConnectionOptions();
    // ...
}
```

### 4.3 Return Values

- Return meaningful types. Use `T?` for values that may be absent.
- Never return `null` from a collection — return empty collections.
- Use `Result<T>` pattern or exceptions for error cases.

### 4.4 Expression-Bodied Members

```csharp
public string FullName => $"{FirstName} {LastName}";

public override string ToString() => $"User({Id}, {FullName})";
```

Use for single-expression properties and methods.

---

## 5. Types & Composition

### 5.1 Records (C# 9+)

```csharp
public record User(string Name, string Email, int Age);

public record OrderItem(string Product, decimal Price, int Quantity)
{
    public decimal Total => Price * Quantity;
}
```

- Use records for immutable data transfer objects and value objects.
- Use `with` expressions for non-destructive mutation.
- Use positional records for simple DTOs, nominal records for complex types.

### 5.2 Structs vs. Classes

- Use **struct** for small, immutable value types (≤16 bytes).
- Use `readonly struct` and `readonly record struct` for value semantics.
- Use **class** when identity matters or the type is large.
- Use `sealed` on classes not designed for inheritance.

### 5.3 Interfaces

```csharp
public interface IUserRepository
{
    Task<User?> FindByIdAsync(int id, CancellationToken ct = default);
    Task<IReadOnlyList<User>> GetAllAsync(CancellationToken ct = default);
    Task SaveAsync(User user, CancellationToken ct = default);
}
```

- Keep interfaces small and focused.
- Use default interface methods (C# 8+) sparingly — prefer explicit
  implementations.

### 5.4 Enums and Pattern Matching

```csharp
public enum OrderStatus
{
    Pending,
    Confirmed,
    Shipped,
    Delivered,
    Cancelled
}

public static decimal CalculateDiscount(OrderStatus status) => status switch
{
    OrderStatus.Confirmed => 0.05m,
    OrderStatus.Shipped => 0.02m,
    _ => 0m
};
```

### 5.5 Discriminated Unions (OneOf / Tagged Unions)

Use the type system to make illegal states unrepresentable:

```csharp
// Note: C# has no Java-style 'sealed permits' to enforce closed hierarchies.
// A private constructor on the base + nested derived records is the standard
// approximation. External code cannot subclass; the `_` arm guards against
// future additions and is required because the compiler cannot prove exhaustiveness.
public abstract record Shape
{
    private Shape() { }  // Prevent external subclassing

    public record Circle(double Radius) : Shape;
    public record Rectangle(double Width, double Height) : Shape;
    public record Triangle(double Base, double Height) : Shape;
}

public static double Area(Shape shape) => shape switch
{
    Shape.Circle(var r) => Math.PI * r * r,
    Shape.Rectangle(var w, var h) => w * h,
    Shape.Triangle(var b, var h) => 0.5 * b * h,
    _ => throw new InvalidOperationException($"Unknown shape: {shape}")
};
```

### 5.6 Dependency Injection

```csharp
public class UserService
{
    private readonly IUserRepository _userRepository;
    private readonly ILogger<UserService> _logger;

    public UserService(IUserRepository userRepository, ILogger<UserService> logger)
    {
        _userRepository = userRepository;
        _logger = logger;
    }
}
```

- Use constructor injection. Register in DI container.
- Use `Microsoft.Extensions.DependencyInjection` or similar.
- Prefer `IOptions<T>` for configuration.

Service lifetimes (`Microsoft.Extensions.DependencyInjection`):

| Lifetime | When to use | Pitfalls |
|---|---|---|
| `Singleton` | Stateless services, expensive-to-build state shared app-wide. | Must be thread-safe. Never capture a `Scoped` dependency. |
| `Scoped` | Per-request services. Default for `DbContext`, repositories, unit-of-work. | Resolving a `Scoped` from a `Singleton` is a captured-dependency bug. |
| `Transient` | Cheap, stateless services. | Never inject a `Transient` `IDisposable` into a `Singleton` — the root container holds it for the app lifetime (memory leak). |

The validator (`ValidateScopes`, `ValidateOnBuild`) catches captive-dependency
bugs at startup — enable both in non-production environments.

### 5.7 Typed Data Passing

Pass structured, typed objects between methods and layers — never
`Dictionary<string, object>`, raw strings, or loosely typed containers.

```csharp
// Bad — untyped, error-prone
public void CreateUser(Dictionary<string, object> data)
{
    var name = (string)data["Name"];     // runtime cast, no compile-time safety
    var age = (int)data["Age"];
}

// Good — typed record
public record CreateUserRequest(string Name, string Email, int Age);

public void CreateUser(CreateUserRequest request)
{
    // compile-time safety, IDE support, self-documenting
}
```

- Define records or DTOs for all data that crosses layer or project
  boundaries.
- At system boundaries (API payloads, config files, CLI args), deserialize
  into typed objects immediately using `System.Text.Json` or the
  appropriate deserializer.
- Use strong typing and generics to encode data contracts.
- Never use `dynamic` for structured data exchange.
- Return typed results, not tuples of loosely related values — use records
  instead.

```csharp
// Bad — tuple of loosely related values
public (string, int, bool) GetUserStatus(int id) { ... }

// Good — self-documenting record
public record UserStatus(string Name, int LoginCount, bool IsActive);
public UserStatus GetUserStatus(int id) { ... }
```

### 5.8 SOLID in C#

| Principle | C# Application |
|---|---|
| **S** — Single Responsibility | One class per concern. |
| **O** — Open/Closed | Extend via interfaces and new implementations. |
| **L** — Liskov Substitution | Subtypes must honour the interface contract. |
| **I** — Interface Segregation | Keep interfaces small and focused. |
| **D** — Dependency Inversion | Accept interfaces, not concrete types. Inject via constructor. |

---

## 6. Documentation & Comments

### 6.1 XML Doc Comments

Use `///` for documentation comments on all public members:

```csharp
/// <summary>
/// Retrieve a user by their unique identifier.
/// </summary>
/// <param name="userId">The unique identifier for the user.</param>
/// <param name="includeDeleted">
/// If <c>true</c>, soft-deleted users are also returned.
/// </param>
/// <returns>The matching user, or <c>null</c> if not found.</returns>
/// <exception cref="DatabaseException">
/// Thrown when the database connection fails.
/// </exception>
public async Task<User?> FindUserAsync(int userId, bool includeDeleted = false)
{
    // ...
}
```

- Document all `public` and `protected` members.
- Use `<summary>`, `<param>`, `<returns>`, `<exception>`, `<remarks>`,
  `<example>`, `<see cref=""/>`.

### 6.2 Comments

- Comments explain **why**, not what.
- Never commit commented-out code.
- Use `// TODO:` and `// FIXME:` sparingly.

---

## 7. Error Handling

### 7.1 Principles

```csharp
public class NotFoundException : Exception
{
    public string Entity { get; }
    public object Id { get; }

    public NotFoundException(string entity, object id)
        : base($"{entity} with id {id} not found")
    {
        Entity = entity;
        Id = id;
    }
}
```

- Use exceptions for exceptional situations, not control flow.
- Create custom exception types for domain errors.
- Derive from `Exception` (not `ApplicationException`).

### 7.2 Try/Catch

```csharp
try
{
    var user = await FindUserAsync(id);
}
catch (NotFoundException ex) when (ex.Entity == "User")
{
    _logger.LogWarning("User {Id} not found", id);
}
catch (DatabaseException ex)
{
    _logger.LogError(ex, "Database failure");
    throw;
}
```

- Catch specific exception types. Never catch `Exception` at the top level
  without re-throwing.
- Use exception filters (`when`) for conditional catching.
- Use `throw;` (not `throw ex;`) to preserve stack trace.

### 7.3 Resource Cleanup

Use `using` declarations and `IDisposable`/`IAsyncDisposable`:

```csharp
await using var connection = new SqlConnection(connectionString);
await connection.OpenAsync();

using var reader = await command.ExecuteReaderAsync();
while (await reader.ReadAsync())
{
    // process
}
```

---

## 8. Namespaces & Imports

### 8.1 Namespace Design

- Match namespace to folder structure: `MyApp.Services` in `Services/` folder.
- Use file-scoped namespaces (C# 10+).
- Do not use deeply nested namespaces — 3–4 levels maximum.

### 8.2 Using Directive Ordering

1. `System.*` namespaces
2. `Microsoft.*` namespaces
3. Third-party namespaces
4. Project namespaces

Place `using` directives outside the namespace declaration (or at file top
with file-scoped namespaces). Sort alphabetically within each group.

### 8.3 Global Usings (C# 10+)

```csharp
// GlobalUsings.cs
global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading.Tasks;
```

### 8.4 One Library per Concern

| Overlapping libraries | Pick one |
|---|---|
| `HttpClient` / RestSharp / Refit | One HTTP client |
| `System.Text.Json` / Newtonsoft.Json | One JSON library |
| EF Core / Dapper / ADO.NET | One data access library |
| NLog / Serilog / log4net | One logging library |
| xUnit / NUnit / MSTest | One test framework |

---

## 9. Design Patterns in C#

### 9.1 Creational

- **Factory**: static factory methods or `IFactory<T>` interface.
- **Builder**: fluent API with method chaining.
- **Singleton**: DI container (`AddSingleton<T>`), not static instance.

### 9.2 Structural

- **Decorator**: wrapping an interface implementation with additional behaviour.
- **Adapter**: implementing an interface by delegating to an incompatible type.
- **Facade**: simplified API over complex subsystem.

### 9.3 Behavioural

- **Strategy**: interface + DI.
- **Observer**: events, `IObservable<T>`, or MediatR notifications.
- **Template Method**: abstract base class with overridable steps.
- **Mediator**: MediatR or custom `IMediator`.

---

## 10. Testing

### 10.1 Principles

- Use **xUnit** (recommended), **NUnit**, or **MSTest**. One per project.
- **Test behaviour, not implementation.** Tests should survive refactors.
- Testing pyramid: unit > integration > E2E.

### 10.2 Naming

```csharp
// MethodName_Scenario_ExpectedResult
[Fact]
public async Task Withdraw_InsufficientFunds_ThrowsException()
```

### 10.3 Structure (Arrange-Act-Assert)

```csharp
[Fact]
public void ApplyDiscount_ReducesTotal()
{
    // Arrange
    var cart = new ShoppingCart(new[] { new Item("Book", 20.0m) });
    var discount = new PercentageDiscount(10);

    // Act
    cart.Apply(discount);

    // Assert
    Assert.Equal(18.0m, cart.Total);
}
```

### 10.4 Mocking

- Use **Moq**, **NSubstitute**, or **FakeItEasy**.
- Mock interfaces at system boundaries: repositories, HTTP clients, clocks.
- Use `ITimeProvider` (or `TimeProvider` in .NET 8+) for time-dependent tests.

### 10.5 Integration Testing

```csharp
public class UserApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public UserApiTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task GetUser_ReturnsOk()
    {
        var response = await _client.GetAsync("/api/users/1");
        response.EnsureSuccessStatusCode();
    }
}
```

---

## 11. Database Access & ACID

### 11.1 Entity Framework Core

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<User> Users => Set<User>();
    public DbSet<Order> Orders => Set<Order>();
}
```

The `DbContextOptions<TContext>` constructor is required when registering
`DbContext` via `AddDbContext<AppDbContext>(...)` — without it EF Core
cannot inject the configured options.

### 11.2 Transactions

```csharp
await using var transaction = await dbContext.Database.BeginTransactionAsync();
try
{
    fromAccount.Balance -= amount;
    toAccount.Balance += amount;
    await dbContext.SaveChangesAsync();
    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

### 11.3 Parameterised Queries (Dapper / Raw SQL)

```csharp
// Yes — parameters
var user = await connection.QuerySingleOrDefaultAsync<User>(
    "SELECT * FROM Users WHERE Email = @Email",
    new { Email = email });

// No — interpolation (SQL injection). Dapper's QueryAsync takes a string,
// so the value of `email` is concatenated into the SQL before Dapper ever
// sees it.
var users = await connection.QueryAsync<User>(
    $"SELECT * FROM Users WHERE Email = '{email}'");
```

### 11.4 SQL Injection Protection

Always use parameterised queries. Never build SQL from user input via
string interpolation or concatenation.

```csharp
// ===== BAD — SQL injection vulnerability =====

// String interpolation — attacker controls the query
var sql = $"SELECT * FROM Users WHERE Email = '{email}'";

// String concatenation — equally dangerous
var sql = "SELECT * FROM Users WHERE Email = '" + email + "'";

// FromSqlRaw with concatenation
var users = db.Users.FromSqlRaw("SELECT * FROM Users WHERE Name = '" + name + "'");


// ===== GOOD — parameterised and safe =====

// ADO.NET — @ParamName placeholders
using var cmd = new SqlCommand("SELECT * FROM Users WHERE Email = @Email", connection);
cmd.Parameters.AddWithValue("@Email", email);

// Dapper — anonymous object parameters
var user = await connection.QuerySingleOrDefaultAsync<User>(
    "SELECT * FROM Users WHERE Email = @Email",
    new { Email = email });

// EF Core LINQ — parameterised automatically
var user = await db.Users.FirstOrDefaultAsync(u => u.Email == email);

// EF Core raw SQL — FromSqlInterpolated (safe, not FromSqlRaw)
var users = await db.Users
    .FromSqlInterpolated($"SELECT * FROM Users WHERE Email = {email}")
    .ToListAsync();
```

- Always use `@ParamName` placeholders with command parameters.
- Never use `$"...{value}..."` or `"..." + value` to build SQL strings.
- EF Core LINQ queries are parameterised automatically — prefer them.
- For raw SQL in EF Core, use `FromSqlInterpolated` (safe) — never
  `FromSqlRaw` with concatenation.
- Dapper: pass parameters via anonymous objects `new { Email = email }`.
- Validate and constrain input before it reaches the database layer.

### 11.5 Connection Lifecycle

- Use DI-managed `DbContext` scoped per request.
- Use connection pooling (built-in with ADO.NET / EF Core).
- Always dispose connections (`using` / `await using`).

---

## 12. Concurrency & Async

### 12.1 Async/Await

```csharp
public async Task<User> GetUserAsync(int id, CancellationToken ct = default)
{
    var user = await _repository.FindByIdAsync(id, ct);
    return user ?? throw new NotFoundException("User", id);
}
```

- Use `async`/`await` for all I/O-bound operations.
- Accept `CancellationToken` on all async public methods.
- Suffix async methods with `Async`.
- Never use `.Result` or `.Wait()` — deadlock risk.
- Use `ConfigureAwait(false)` in library code.
- **Never `async void`** except for event handlers. `async void` swallows
  the returned `Task`, so exceptions cannot be `await`ed or `catch`ed —
  they propagate up to the `SynchronizationContext` and typically crash
  the process. Always return `Task` (or `ValueTask`).

### 12.2 Task Parallelism

```csharp
var usersTask = GetUsersAsync(ct);
var ordersTask = GetOrdersAsync(ct);
await Task.WhenAll(usersTask, ordersTask);
var users = await usersTask;
var orders = await ordersTask;

// Or with WhenAll over a sequence
var tasks = urls.Select(url => FetchAsync(url, ct));
var results = await Task.WhenAll(tasks);
```

### 12.3 Thread Safety

- Use `ConcurrentDictionary<TKey, TValue>` and other concurrent collections.
- Use `SemaphoreSlim` for async locking.
- Use `Channel<T>` for producer-consumer patterns.
- Use `Interlocked` for atomic operations.
- Prefer immutable data to synchronisation.

---

## 13. Performance & Idiomatic C#

### 13.1 LINQ

```csharp
var activeNames = users
    .Where(u => u.IsActive)
    .Select(u => u.Name)
    .OrderBy(name => name)
    .ToList();
```

- Use method syntax for chaining. Query syntax for complex joins.
- Avoid LINQ in hot paths — consider manual loops.
- Use `AsNoTracking()` for read-only EF Core queries.

### 13.2 Pattern Matching (C# 9+)

```csharp
var description = shape switch
{
    Circle { Radius: > 10 } => "Large circle",
    Circle { Radius: > 0 } => "Small circle",
    Rectangle { Width: var w, Height: var h } when w == h => "Square",
    Rectangle => "Rectangle",
    _ => "Unknown"
};
```

### 13.3 Guard Clauses

```csharp
public Receipt ProcessOrder(Order order)
{
    ArgumentNullException.ThrowIfNull(order);
    if (order.IsCancelled) throw new OrderException("Order cancelled");
    if (!order.Items.Any()) return Receipt.Empty;
    return BuildReceipt(order);
}
```

### 13.4 Span and Memory

- Use `Span<T>` and `ReadOnlySpan<T>` for high-performance buffer processing.
- Use `string.AsSpan()` to avoid allocations.
- Use `ArrayPool<T>` for temporary arrays.

### 13.5 Nullable Reference Types

```csharp
// Enable in .csproj: <Nullable>enable</Nullable>

public string Name { get; }            // Non-null
public string? MiddleName { get; }     // Nullable
```

- Enable project-wide. Fix all warnings.
- Use `!` (null-forgiving) only when the compiler cannot prove safety.

---

## 14. Defensive Programming & Input Validation

### 14.1 Validate External Input at Boundaries

Validate all external input at controller and service boundaries. Never
trust data from API requests, form submissions, query strings, file
uploads, or third-party integrations.

### 14.2 Data Annotations

Use Data Annotations for declarative validation on models and DTOs:

```csharp
public record CreateUserRequest
{
    [Required]
    [StringLength(100, MinimumLength = 2)]
    public required string Name { get; init; }

    [Required]
    [EmailAddress]
    public required string Email { get; init; }

    [Range(1, 150)]
    public int Age { get; init; }

    [RegularExpression(@"^\+?[1-9]\d{1,14}$", ErrorMessage = "Invalid phone number")]
    public string? Phone { get; init; }

    [StringLength(255)]
    public string? Bio { get; init; }
}
```

- `[Required]` — rejects null or missing values.
- `[StringLength(max)]` / `[StringLength(max, MinimumLength = min)]` — constrains length.
- `[Range(min, max)]` — constrains numeric values.
- `[RegularExpression(pattern)]` — constrains format.
- `[EmailAddress]`, `[Phone]`, `[Url]` — built-in format validators.

### 14.3 FluentValidation

Use FluentValidation for complex validation rules that go beyond what
Data Annotations can express:

```csharp
public class CreateOrderValidator : AbstractValidator<CreateOrderRequest>
{
    public CreateOrderValidator()
    {
        RuleFor(x => x.Items).NotEmpty().WithMessage("Order must have at least one item");
        RuleFor(x => x.ShippingAddress).NotNull().SetValidator(new AddressValidator());
        RuleForEach(x => x.Items).ChildRules(item =>
        {
            item.RuleFor(i => i.Quantity).GreaterThan(0);
            item.RuleFor(i => i.Price).GreaterThan(0);
        });
    }
}
```

### 14.4 Guard Clauses

Use built-in guard clauses (.NET 8+) at method entry points:

```csharp
public void TransferFunds(Account from, Account to, decimal amount)
{
    ArgumentNullException.ThrowIfNull(from);
    ArgumentNullException.ThrowIfNull(to);
    ArgumentException.ThrowIfNullOrWhiteSpace(from.AccountNumber);
    ArgumentOutOfRangeException.ThrowIfLessThanOrEqual(amount, 0);
    ArgumentOutOfRangeException.ThrowIfGreaterThan(amount, from.Balance);

    // proceed with transfer
}
```

### 14.5 Value Objects with Validation

Use `record` types with validation in constructors for value objects:

```csharp
public sealed record EmailAddress
{
    public string Value { get; }

    public EmailAddress(string value)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(value);
        if (!value.Contains('@'))
            throw new ArgumentException("Invalid email address", nameof(value));
        Value = value;
    }
}
```

Note: using a non-positional `record` with a manual read-only auto-property
(`get;`) is valid — the property is assigned in the constructor. Mark the
record `sealed` to prevent inheritance subverting the validation invariant.
The compiler-generated `Equals`/`GetHashCode` still uses `Value`.

### 14.6 Collection and Dictionary Safety

- Check collection bounds before indexing. Use `TryGetValue` for
  dictionary lookups.
- Use `FirstOrDefault` / `SingleOrDefault` instead of `First` / `Single`
  when absence is possible.

```csharp
// Bad — throws on missing key
var value = dictionary[key];

// Good — safe lookup
if (dictionary.TryGetValue(key, out var value))
{
    // use value
}
```

### 14.7 Dangerous Input Vectors

- Sanitize input used in file paths (`Path.GetFullPath`, `Path.Combine`),
  URLs, or process execution.
- Never pass user input directly to `Process.Start()` without
  sanitization.
- Never use `Type.GetType()` or reflection with unsanitized user input.
- Use nullable reference types to catch null issues at compile time.
- Use `Regex.IsMatch` with a timeout for user-supplied patterns to
  prevent ReDoS attacks.

```csharp
// Bad — path traversal
var filePath = Path.Combine(uploadsDir, userFileName);

// Good — resolve both sides and ensure the path stays within the root
var rootFull = Path.GetFullPath(uploadsDir);
if (!rootFull.EndsWith(Path.DirectorySeparatorChar))
    rootFull += Path.DirectorySeparatorChar;
var fullPath = Path.GetFullPath(Path.Combine(rootFull, userFileName));
if (!fullPath.StartsWith(rootFull, StringComparison.Ordinal))
    throw new ArgumentException("Invalid file path");

// Regex with timeout to prevent ReDoS
var isMatch = Regex.IsMatch(input, pattern, RegexOptions.None,
    TimeSpan.FromMilliseconds(100));
```

---

## 15. Project Structure

### 15.1 Solution Layout

```
MyApp/
    MyApp.sln
    src/
        MyApp.Domain/
            Entities/
            ValueObjects/
            Interfaces/
        MyApp.Application/
            Services/
            DTOs/
        MyApp.Infrastructure/
            Data/
            External/
        MyApp.Api/
            Controllers/
            Program.cs
    tests/
        MyApp.Domain.Tests/
        MyApp.Application.Tests/
        MyApp.Api.Tests/
```

### 15.2 Dependency Management

- Use NuGet for packages. Central Package Management (`Directory.Packages.props`)
  for version consistency.
- Use `Directory.Build.props` for shared project settings.
- Lock file: `packages.lock.json` for reproducible builds.

---

## 16. Tooling

| Purpose | Tool | Notes |
|---|---|---|
| Formatter | **dotnet format** | Built-in, `.editorconfig` |
| Linter | **Roslyn Analyzers** / **StyleCop.Analyzers** | Compile-time checks |
| Test runner | **dotnet test** | xUnit / NUnit / MSTest |
| Test framework | **xUnit** (recommended) | Modern, extensible |
| Mocking | **Moq** / **NSubstitute** | Interface mocking |
| Coverage | **Coverlet** | Cross-platform |
| Static analysis | **SonarAnalyzer** / **Roslynator** | Code quality |
| Build | **dotnet CLI** / **MSBuild** | Cross-platform |
| IDE | **Visual Studio** / **Rider** / **VS Code** | Full C# support |

---

## 17. Build Tools

### 17.1 dotnet CLI (Primary)

Microsoft's unified build tool for .NET:

```bash
dotnet new console  # Create new project
dotnet build  # Compile project
dotnet build -c Release  # Release configuration
dotnet run  # Build and run
dotnet test  # Run tests
dotnet publish -c Release -o ./publish  # Publish for deployment
dotnet pack  # Create NuGet package
dotnet nuget push bin/Release/*.nupkg  # Publish package
```

### 17.2 Project File Configuration

.csproj file controls build, dependencies, and metadata:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Serilog" Version="4.0.0" />
    <PackageReference Include="Dapper" Version="2.1.35" />
  </ItemGroup>

  <Target Name="BuildNumber" BeforeTargets="Build">
    <Exec Command="echo Build: $(BuildId)" />
  </Target>
</Project>
```

### 17.3 MSBuild (Advanced)

Lower-level build engine (used by dotnet CLI):

```bash
msbuild MyProject.csproj  # Direct MSBuild invocation
msbuild MyProject.csproj /p:Configuration=Release
```

Custom targets and properties for complex builds.

### 17.4 Solution Files

Manage multiple projects:

```bash
dotnet sln add MyProject/MyProject.csproj
dotnet sln MyApp.sln list
dotnet build MyApp.sln  # Build all projects
```

### 17.5 Docker for .NET Applications

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS builder
WORKDIR /app
COPY *.csproj ./
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/runtime:8.0
WORKDIR /app
COPY --from=builder /app/publish ./
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

### 17.6 CI/CD Integration

GitHub Actions:
```yaml
name: Build & Test
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-dotnet@v3
        with: { dotnet-version: '8.0' }
      - run: dotnet build
      - run: dotnet test
```

---

## 18. SBOM Creation

### 18.1 What is an SBOM?

A Software Bill of Materials documents all NuGet packages and transitive dependencies. Critical for vulnerability tracking, license compliance, and supply chain security.

### 18.2 NuGet Dependency Management

Dependencies declared in `.csproj`:

```xml
<ItemGroup>
    <PackageReference Include="Serilog" Version="4.0.0" />
    <PackageReference Include="Dapper" Version="2.1.35" />
</ItemGroup>
```

Lock file (`packages.lock.json`):

```bash
dotnet build --use-lock-file  # Creates packages.lock.json
```

### 18.3 SBOM Generation with CycloneDX

**Using `CycloneDX.dotnet`**:

```bash
dotnet tool install --global CycloneDX
CycloneDX --project-path MyApp.csproj --output sbom.json
CycloneDX --project-path MyApp.csproj --output sbom.xml --format xml
```

Or integrate into build:

```xml
<Target Name="GenerateSBOM" AfterTargets="Build">
    <Exec Command="CycloneDX --project-path $(MSBuildProjectFullPath)" />
</Target>
```

### 18.4 Vulnerability Scanning

**Using `dotnet list package --vulnerable`**:

```bash
dotnet list package --vulnerable  # Shows known vulns
dotnet list package --outdated    # Shows outdated packages
```

**GitHub Dependabot** (native integration):

- Scans `.csproj` and `packages.lock.json`
- Creates PRs for updates
- Dashboard on Security tab

**Snyk CLI**:

```bash
snyk test --file=packages.lock.json
```

### 18.5 License Compliance

**Using `dotnet-project-licenses`**:

```bash
dotnet tool install --global dotnet-project-licenses
dotnet-project-licenses --input MyProject.csproj --json > licenses.json
```

NuGet packages should declare licenses; verify compliance with corporate policy.

### 18.6 Integration into CI/CD

- Include `packages.lock.json` in VCS (required for reproducibility)
- Run `dotnet list package --vulnerable` on every PR
- Generate SBOM with CycloneDX on release
- Verify license compliance; fail on non-whitelisted licenses
- Use GitHub Dependabot or Snyk for continuous monitoring
- Store SBOM and audit reports as release artifacts

---

## 19. References

### Official Documentation

| Resource | URL |
|---|---|
| C# Documentation | https://learn.microsoft.com/en-us/dotnet/csharp/ |
| C# Coding Conventions | https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions |
| .NET API Design Guidelines | https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/ |
| .NET Documentation | https://learn.microsoft.com/en-us/dotnet/ |

### Books

| Book | Authors | Key Takeaways |
|---|---|---|
| *C# in Depth* | Jon Skeet (2019) | Deep language mechanics — generics, async, LINQ, nullable. |
| *Effective C#* | Bill Wagner (2017) | 50 items for better C# — idioms, performance, API design. |
| *Concurrency in C# Cookbook* | Stephen Cleary (2019) | Async/await patterns, parallel programming, reactive extensions. |
| *Design Patterns* | Gamma et al. (1994) | Composition over inheritance; program to interfaces. |
| *Clean Code* | Robert C. Martin (2008) | Small functions, meaningful names, SRP. |
| *Clean Architecture* | Robert C. Martin (2017) | Dependency inversion, layered architecture. |
