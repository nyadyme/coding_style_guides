# C# — AI Coding Rules

Apply these rules when generating or reviewing C# code.

## Boilerplate

- Enable `<Nullable>enable</Nullable>` project-wide.
- File-scoped namespaces (C# 10+): `namespace MyApp.Services;`.
- Global usings for common namespaces in `GlobalUsings.cs`.
- Target .NET 8+ when possible.

## Formatting

- 4 spaces indentation. Never tabs.
- 120 characters recommended line limit.
- Allman braces (opening brace on its own line).
- One blank line between method definitions.
- No blank line after opening brace or before closing brace.
- One blank line between `using` directive groups.
- Use `dotnet format` and `.editorconfig` for enforcement.

## Naming

- `PascalCase` for classes, structs, records, interfaces, methods, properties, events, enums, enum members, constants, namespaces.
- `IPascalCase` for interfaces (prefix `I`).
- `camelCase` for local variables and parameters.
- `_camelCase` for private fields.
- `TPascalCase` for type parameters (prefix `T`).
- `Async` suffix on async methods.
- Boolean: prefix `Is`, `Has`, `Can`.
- No Hungarian notation. No member prefixes other than `_` for private fields.
- Two-letter acronyms all caps (`IO`), three+ PascalCase (`Xml`, `Json`).
- File name matches primary type.

## Methods

- Small (under 20 lines), one thing, one level of abstraction.
- Max 3-4 parameters. Group into record or class if more.
- Return `T?` for values that may be absent. Never return `null` from collections.
- Expression-bodied members for single-expression properties and methods.
- Use `var` when the type is obvious from the right-hand side.

## Types & Composition

- Records (C# 9+) for immutable data. `with` expressions for non-destructive mutation.
- `readonly struct` / `readonly record struct` for small value types.
- `sealed` on classes not designed for inheritance.
- Interfaces over abstract classes. Keep interfaces small and focused.
- Enums for fixed sets. Pattern matching with `switch` expressions.
- Abstract record hierarchies for discriminated unions.
- Constructor injection for dependency injection. Register in DI container.
- Pass typed objects (records, DTOs) between methods and layers — never `Dictionary<string, object>`, raw strings, or `dynamic`.
- Define records or DTOs for all data crossing layer/project boundaries. Deserialize external input into typed objects immediately.
- Return typed results, not tuples of loosely related values — use records instead.

## SOLID

- **S**: One class per concern.
- **O**: Extend via new interface implementations.
- **L**: Subtypes must honour the interface contract.
- **I**: Keep interfaces small and focused.
- **D**: Accept interfaces, not concrete types. Inject via constructor.

## Documentation (XML Doc Comments)

- `///` on all `public` and `protected` members.
- `<summary>`, `<param>`, `<returns>`, `<exception>`, `<remarks>`.
- First sentence is the summary.

## Error Handling

- Custom exceptions extending `Exception` (not `ApplicationException`).
- Catch specific types. Exception filters with `when`.
- `throw;` (not `throw ex;`) to preserve stack trace.
- Never catch `Exception` broadly without re-throwing.
- `using` / `await using` for `IDisposable` / `IAsyncDisposable` resources.
- Never swallow exceptions silently.

## Imports

- Order: `System.*`, `Microsoft.*`, third-party, project namespaces. Alphabetical within groups.
- Global usings (C# 10+) for common namespaces.
- One library per concern: one HTTP client, one JSON library, one ORM, one logging library, one test framework.

## Concurrency & Async

- `async`/`await` for all I/O-bound operations.
- `CancellationToken` on all async public methods.
- Suffix async methods with `Async`.
- Never `.Result` or `.Wait()` — deadlock risk.
- `ConfigureAwait(false)` in library code.
- `Task.WhenAll` for concurrent operations.
- `ConcurrentDictionary`, `SemaphoreSlim`, `Channel<T>` for thread safety.
- Prefer immutable data over synchronisation.
- Never `async void` except for event handlers — exceptions cannot be caught and propagate to `SynchronizationContext`/process. Always return `Task` (or `ValueTask`).
- Prefer `await using` over `using` for `IAsyncDisposable`. Implementing both `IDisposable` and `IAsyncDisposable` is common; use the async form in async code paths so `DisposeAsync` runs.

## Database / ACID

- EF Core or Dapper — one data access library per project.
- Explicit transactions. Rollback on failure.
- Parameterised queries only. Never string interpolation or concatenation for SQL.
- Always use `@ParamName` placeholders. Dapper: `new { Email = email }` for parameters.
- EF Core LINQ is auto-parameterised — prefer it. For raw SQL, use `FromSqlInterpolated` (safe), never `FromSqlRaw` with concatenation.
- Validate and constrain input before it reaches the database layer.
- `using` / `await using` for connections.
- `AsNoTracking()` for read-only EF Core queries — disables change tracking, reduces memory, and avoids accidental updates.
- `DbContext` is scoped per request (`AddDbContext` registers it as Scoped) and is **not** thread-safe — never share an instance across concurrent operations or `await Task.WhenAll` of queries on the same context.
- `SaveChangesAsync` runs inside an implicit transaction for a single call; use `Database.BeginTransactionAsync` only when multiple `SaveChangesAsync` calls or raw SQL must be atomic together.

## Testing

- xUnit (recommended), NUnit, or MSTest. One per project.
- `MethodName_Scenario_ExpectedResult` naming.
- Arrange-Act-Assert structure.
- Moq, NSubstitute, or FakeItEasy for mocking interfaces at system boundaries.
- `WebApplicationFactory<Program>` for integration tests.
- `TimeProvider` (.NET 8+) for time-dependent tests.

## Performance & Idioms

- LINQ method syntax for chaining. Query syntax for complex joins.
- Pattern matching (C# 9+): property patterns, type patterns, relational patterns.
- Guard clauses with `ArgumentNullException.ThrowIfNull()`.
- `Span<T>` / `ReadOnlySpan<T>` for high-performance buffer processing.
- Nullable reference types: fix all warnings. Use `!` only when compiler cannot prove safety.

## Patterns

- Factory: static factory methods or `IFactory<T>`.
- Builder: fluent API with method chaining.
- Singleton: DI container (`AddSingleton<T>`), not static instance.
- Strategy: interface + DI. Observer: events or MediatR.
- Mediator: MediatR or custom `IMediator`.

## DI Lifetimes

- `Singleton`: one instance for the application lifetime. Must be thread-safe. Never capture a `Scoped` dependency from a Singleton — captures a stale scope.
- `Scoped`: one instance per request (ASP.NET Core) or per `IServiceScope`. Default for `DbContext`, repositories, unit-of-work services.
- `Transient`: a fresh instance every resolve. Use for stateless, cheap-to-construct services. Never inject a `Transient` `IDisposable` into a `Singleton` — the container will hold it for the app lifetime.

## Defensive Programming & Input Validation

- Validate all external input at controller/service boundaries using Data Annotations (`[Required]`, `[StringLength]`, `[Range]`, `[RegularExpression]`, `[EmailAddress]`) or FluentValidation.
- Guard clauses: `ArgumentNullException.ThrowIfNull()`, `ArgumentException.ThrowIfNullOrWhiteSpace()`, `ArgumentOutOfRangeException.ThrowIfLessThan()` (.NET 8+).
- Use `record` types with constructor validation for value objects.
- `TryGetValue` for dictionary lookups. `FirstOrDefault` when absence is possible.
- Sanitize input used in file paths (`Path.GetFullPath`), URLs, or process execution. Never pass unsanitized user input to `Process.Start()`, `Type.GetType()`, or reflection.
- Enable nullable reference types. Fix all warnings.
- `Regex.IsMatch` with timeout for user-supplied patterns (prevent ReDoS).

## Project Structure

- Solution with `src/` and `tests/` directories.
- Layered: `Domain`, `Application`, `Infrastructure`, `Api` projects.
- NuGet for packages. Central Package Management (`Directory.Packages.props`).
- `Directory.Build.props` for shared settings.

## Tooling

- dotnet format (formatter), Roslyn Analyzers / StyleCop.Analyzers (linters).
- xUnit (testing), Coverlet (coverage), dotnet CLI (build).
- SonarAnalyzer / Roslynator (static analysis).

## Build Tools

- dotnet CLI is the primary build tool: `dotnet build`, `dotnet test`, `dotnet publish`.
- Project files (.csproj) control build, dependencies, and properties.
- Release builds: `dotnet build -c Release`.
- Create NuGet packages: `dotnet pack`, publish with `dotnet nuget push`.
- MSBuild for advanced/custom builds.
- Docker multi-stage builds for .NET applications.

## SBOM Creation

- Declare NuGet dependencies in `.csproj` with pinned versions.
- Include `packages.lock.json` in VCS for reproducibility.
- Use `dotnet list package --vulnerable` to detect known vulns.
- Use `dotnet-project-licenses` for license compliance.
- GitHub Dependabot or Snyk for continuous vulnerability scanning.
- Gate CI/CD on clean vulnerability reports.
