# Java — AI Coding Rules

Apply these rules when generating or reviewing Java code.

## Boilerplate

- Minimum Java 17. Prefer Java 21+ for records, sealed classes, pattern matching, virtual threads.
- `package` declaration matching directory structure.
- No wildcard imports (`*`). Explicit imports only.

## Formatting

- 4 spaces indentation. Never tabs.
- 100 characters line limit.
- K&R braces (opening brace on same line).
- One blank line between methods.
- One blank line between field groups and method groups.
- No blank line after opening brace or before closing brace.
- Use google-java-format or Checkstyle for enforcement.

## Naming

- `PascalCase` for classes, interfaces, enums, records, annotations.
- `camelCase` for methods, variables, parameters.
- `UPPER_SNAKE_CASE` for `static final` constants.
- `T`, `E`, `K`, `V` or descriptive `PascalCase` for type parameters.
- Boolean: prefix `is`, `has`, `can`, `should`.
- File name matches public type name.
- No Hungarian notation.

## Methods

- Small (under 20 lines), one thing, one level of abstraction.
- Max 3-4 parameters. Group into record or parameter object if more.
- Return `Optional<T>` for values that may be absent. Never return `null` from collections.
- Use `var` (Java 10+) when the type is obvious from the right-hand side.

## Types & Composition

- Prefer records (Java 16+) for immutable data carriers.
- Use sealed classes/interfaces (Java 17+) for closed type hierarchies.
- Prefer interfaces over abstract classes.
- Use enums for fixed sets. Enums can have methods and fields.
- Use pattern matching with switch (Java 21+) for type-safe dispatch.
- `final` on classes not designed for inheritance.
- Dependency injection via constructor. Prefer constructor injection over field injection.
- Pass typed objects (records, classes) between methods — not `Map<String, Object>` or raw strings.
- Define records or classes for all data crossing package boundaries.
- At system boundaries (API responses, config files, CLI args), deserialise into typed objects immediately (Jackson, Gson).
- Use strong typing and generics to encode data contracts.

## SOLID

- **S**: One class per concern.
- **O**: Extend via new interface implementations.
- **L**: Subtypes must honour the interface contract.
- **I**: Keep interfaces small and focused.
- **D**: Accept interfaces, not concrete types. Inject via constructor.

## Documentation (Javadoc)

- `/** */` on all public classes, methods, and fields.
- `@param`, `@return`, `@throws` tags. First sentence is the summary.
- No Javadoc on overrides unless adding information.

## Error Handling

- Prefer unchecked exceptions for programming errors.
- Checked exceptions only when the caller can meaningfully recover.
- Custom exception hierarchy extending `RuntimeException` or `Exception`.
- Never catch `Exception` or `Throwable` broadly without re-throwing.
- Use try-with-resources for all `AutoCloseable` resources.
- Never swallow exceptions silently.

## Imports & Dependencies

- Order: `java.*`, `javax.*`, third-party, project. Alphabetical within groups.
- No wildcard imports.
- One library per concern: one HTTP client, one JSON library, one ORM, one logging facade (SLF4J).

## Concurrency

- Virtual threads (Java 21+) for I/O-bound concurrency.
- `ExecutorService` and `CompletableFuture` for task parallelism.
- Immutable objects over synchronisation.
- `ConcurrentHashMap`, `AtomicReference` for thread-safe state.
- Never `synchronized` on public methods — use private lock objects.
- On Java 21, prefer `ReentrantLock` over `synchronized` inside code that may run on a virtual thread (synchronized pins the carrier thread). JDK 24 removed this pinning.

## Database / ACID

- Parameterised queries only. Never string concatenation for SQL.
- Explicit transaction boundaries. Rollback on failure.
- Use connection pools (HikariCP).
- Try-with-resources for connections, statements, result sets.
- JPA/Hibernate or JDBC — one data access approach per project.
- Always use `PreparedStatement` with `?` placeholders. Never `Statement` with concatenation.
- Never use `String.format` or `+` to build SQL queries.
- Use JPA/Hibernate named parameters (`:param`) or `CriteriaBuilder` for dynamic queries.
- Validate and constrain input before it reaches the database layer.

## Testing

- JUnit 5 + AssertJ (assertions) + Mockito (mocking).
- `test_MethodName_Scenario_ExpectedResult` or `@DisplayName`.
- Arrange-Act-Assert structure.
- Mock at system boundaries: repositories, HTTP clients, clocks.
- `@ParameterizedTest` for data-driven tests.

## Patterns

- Factory: static factory methods (`of()`, `from()`, `create()`).
- Builder: static inner `Builder` class or Lombok `@Builder`.
- Singleton: DI container, not `static` instance.
- Strategy: interface + DI. Observer: events or listeners.

## Defensive Programming

- Validate all external input at controller/service boundaries.
- Use Bean Validation annotations: `@NotNull`, `@NotBlank`, `@Size`, `@Min`, `@Max`, `@Pattern`.
- Validate method arguments: `Objects.requireNonNull()`, Guava `Preconditions`.
- Validate string lengths and numeric ranges explicitly when annotations are not applicable.
- Use enums and sealed types to constrain domains.
- Check collection bounds before indexing. Use `Optional` to avoid null dereferences.
- Sanitize input in file paths (`Path.normalize()`, `Path.toRealPath()`).
- Never pass user input to `Runtime.exec()` without sanitization.
- Never use `Class.forName()` or reflection with unsanitized user input.
- Use records with compact constructors for validated value objects.

## Project Structure

- Maven or Gradle. Standard `src/main/java`, `src/test/java` layout.
- Package by feature, not by layer.
- `module-info.java` for Java modules (JPMS) when applicable.

## Tooling

- google-java-format (formatter), Checkstyle / SpotBugs (linters).
- JUnit 5 (testing), JaCoCo (coverage), Maven/Gradle (build).

## SBOM Creation

- Use CycloneDX Maven or Gradle plugins for SBOM generation.
- Generate on every release; store alongside binaries.
- Scan for vulnerabilities with `dependency-check-maven` (OWASP) or the `org.owasp.dependencycheck` Gradle plugin.
- Verify license compliance with `license-maven-plugin` (from `org.codehaus.mojo`).
- Output formats: JSON or XML (CycloneDX/SPDX).
- Include all transitive dependencies and frameworks.
