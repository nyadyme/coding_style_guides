# Java Coding Style Guidelines

A comprehensive guide rooted in the Google Java Style Guide, Oracle's Code
Conventions, *Effective Java* (Bloch, 2018), and established software
engineering literature — notably *Design Patterns* (Gamma et al., 1994) and
*Clean Code* (Martin, 2008).

---

## Table of Contents

1. [Philosophy](#1-philosophy)
2. [Code Layout & Formatting](#2-code-layout--formatting)
3. [Naming Conventions](#3-naming-conventions)
4. [Methods](#4-methods)
5. [Classes, Interfaces & Composition](#5-classes-interfaces--composition)
6. [Documentation & Comments](#6-documentation--comments)
7. [Error Handling](#7-error-handling)
8. [Packages & Imports](#8-packages--imports)
9. [Design Patterns in Java](#9-design-patterns-in-java)
10. [Testing](#10-testing)
11. [Database Access & ACID](#11-database-access--acid)
12. [Concurrency](#12-concurrency)
13. [Performance & Idiomatic Java](#13-performance--idiomatic-java)
14. [Defensive Programming & Input Validation](#14-defensive-programming--input-validation)
15. [Project Structure](#15-project-structure)
16. [Tooling](#16-tooling)
17. [SBOM Creation](#17-sbom-creation)
18. [References](#18-references)

---

## 1. Philosophy

### 1.1 Guiding Principles

| Principle | Source | Summary |
|---|---|---|
| **Readability over cleverness** | *Clean Code* Ch. 1, Google Style | Code is read far more than written. Optimise for the reader. |
| **Favour composition over inheritance** | *Effective Java* Item 18, *Design Patterns* Ch. 1 | Prefer composition and interfaces over extending concrete classes. |
| **Program to interfaces, not implementations** | *Design Patterns* Ch. 1, *Effective Java* Item 64 | Depend on abstractions. Use interface types for variables. |
| **Minimise mutability** | *Effective Java* Item 17 | Make classes immutable unless there is a good reason not to. |
| **Single Responsibility** | *Clean Code* Ch. 10, SOLID | Every class and method should have one reason to change. |
| **YAGNI** | XP / *Clean Code* | Do not build for hypothetical future requirements. |
| **Least Surprise** | General SE | APIs should behave the way a reasonable developer would expect. |

---

## 2. Code Layout & Formatting

### 2.1 Formatter

All Java code **must** be formatted with **google-java-format** or an
equivalent enforced via CI. No manual formatting debates.

### 2.2 Indentation

- Use **2 spaces** per indentation level (Google style) or **4 spaces**
  (Oracle/IntelliJ default). Pick one and enforce consistently.
- Never use tabs.
- Continuation lines are indented by **4 spaces** (minimum).

### 2.3 Line Length

- **100 characters** maximum (Google style). Teams may use **120**.
- Break long lines at logical points: after commas, before operators,
  after opening parentheses.

### 2.4 Blank Lines

- **One blank line** between method definitions.
- **One blank line** between field declarations and the first method.
- **One blank line** between logical sections within a method.
- **Two blank lines** before the first member of a class and after the last.
- **No blank line** after the opening brace or before the closing brace of a
  method, class, or block.
- **One blank line** between import groups (java, javax, third-party, project).
- **No trailing blank lines** at the end of a file.

### 2.5 Braces

Use K&R style (opening brace on same line):

```java
if (condition) {
    doSomething();
} else {
    doOther();
}

for (int i = 0; i < n; i++) {
    process(i);
}
```

- Always use braces, even for single-statement blocks.

### 2.6 File Organisation

1. License/copyright header (if required)
2. `package` statement
3. `import` statements (no wildcards)
4. Exactly one top-level class per file
5. Class members ordered: static fields, instance fields, constructors,
   methods (public before private)

---

## 3. Naming Conventions

| Entity | Convention | Example |
|---|---|---|
| Package | `all.lowercase.dotted` | `com.example.auth` |
| Class / Interface | `PascalCase` | `UserService`, `Serializable` |
| Enum | `PascalCase`, constants `UPPER_SNAKE_CASE` | `Color.DARK_BLUE` |
| Method | `camelCase` | `calculateTotal()` |
| Variable | `camelCase` | `totalCount` |
| Constant (`static final`) | `UPPER_SNAKE_CASE` | `MAX_RETRIES` |
| Type parameter | Single uppercase or short `PascalCase` | `T`, `K`, `V`, `E` |
| Annotation | `PascalCase` | `@Override`, `@Nullable` |

### 3.1 Naming Guidance (*Clean Code* Ch. 2, *Effective Java*)

- **Use intention-revealing names.** `elapsedTimeInMs` beats `t`.
- **No Hungarian notation.** No `strName`, `iCount`, `m_field`.
- **No single-letter names** except for loop indices and lambdas (`i`, `x`).
- **Boolean methods** read as assertions: `isEmpty()`, `hasPermission()`, `canRead()`.
- **Factory methods**: `of()`, `from()`, `valueOf()`, `create()`, `newInstance()`.
- **Accessor methods**: `getName()`, `setName()`, `isActive()`.

---

## 4. Methods

### 4.1 Size and Focus (*Clean Code* Ch. 3)

- Methods should be **small** — ideally under 20 lines.
- Each method does **one thing** at **one level of abstraction**.
- Extract nested logic into well-named helper methods.

### 4.2 Parameters

- **Fewer parameters are better.** Beyond three, use a parameter object or
  builder.
- **No boolean (flag) arguments** — split into two methods.
- Use **varargs** sparingly and only as the last parameter.

### 4.3 Return Values

- Prefer returning **`Optional<T>`** (Java 8+) over returning `null`.
- Return **unmodifiable collections** from getters:
  `Collections.unmodifiableList(list)` or `List.copyOf(list)`.
- Return consistent types. Avoid mixed return paths.

### 4.4 Overloading

- Use overloading only when the methods perform the same operation on
  different input types.
- Prefer static factory methods over constructor overloading (*Effective Java*
  Item 1).

---

## 5. Classes, Interfaces & Composition

### 5.1 Class Design (*Clean Code* Ch. 10, *Effective Java*)

- Classes should be **small** — measured in responsibilities.
- **High cohesion**: every method should relate to the class's purpose.
- **Minimise accessibility**: use the most restrictive access level possible.
- Prefer `final` classes unless designed for inheritance.

### 5.2 Immutability (*Effective Java* Item 17)

- Make classes **immutable** by default:
  - All fields `private final`.
  - No setters.
  - Return defensive copies of mutable fields.
  - Class itself is `final` (or all constructors private).

```java
public final class Coordinate {
    private final double latitude;
    private final double longitude;

    public Coordinate(double latitude, double longitude) {
        this.latitude = latitude;
        this.longitude = longitude;
    }

    public double latitude() { return latitude; }
    public double longitude() { return longitude; }
}
```

### 5.3 Records (Java 16+)

Use records for immutable data carriers:

```java
public record Coordinate(double latitude, double longitude) {}
```

- Records auto-generate `equals()`, `hashCode()`, `toString()`, and accessors.
- Use compact constructors for validation.

### 5.4 Sealed Classes (Java 17+)

Use `sealed` to restrict class hierarchies:

```java
public sealed interface Shape permits Circle, Rectangle, Triangle {}
public record Circle(double radius) implements Shape {}
public record Rectangle(double width, double height) implements Shape {}
public record Triangle(double base_, double height) implements Shape {}
```

### 5.5 Interfaces vs. Abstract Classes

- **Interfaces** for defining contracts. Prefer over abstract classes.
- **Abstract classes** only when shared implementation is needed.
- Use default methods in interfaces sparingly — prefer composition.
- No `I` prefix on interfaces (`UserService`, not `IUserService`).

### 5.6 Composition over Inheritance (*Effective Java* Item 18)

- Use inheritance only for genuine **is-a** relationships.
- Prefer **composition** (has-a) and **delegation** for code reuse.
- Use **dependency injection** through constructors.

### 5.7 SOLID Principles

| Principle | Java Application |
|---|---|
| **S** — Single Responsibility | One class per concern. |
| **O** — Open/Closed | Extend via new implementations, strategy pattern — not editing existing code. |
| **L** — Liskov Substitution | Subtypes must honour the base type's contract. |
| **I** — Interface Segregation | Keep interfaces small. Don't force clients to depend on methods they don't use. |
| **D** — Dependency Inversion | Depend on interfaces, not concrete classes. Inject via constructors. |

### 5.8 Typed Data Passing

Pass **typed objects** (records, classes) between methods and modules — not
`Map<String, Object>` or raw strings. Loosely-typed maps hide contracts,
resist refactoring, and bypass the compiler's ability to catch errors.

- **Define records or classes** for all data that crosses package boundaries.
- **At system boundaries** (API responses, config files, CLI arguments),
  deserialize into typed objects immediately (Jackson, Gson).
- **Use strong typing and generics** to encode data contracts.

```java
// Bad — Map<String, Object> travels through the application
public Map<String, Object> fetchWeather(String city) {
    var json = httpClient.get("https://api.weather.example/" + city);
    return objectMapper.readValue(json, new TypeReference<>() {});
}

var data = fetchWeather("London");
var temp = (Double) data.get("temperature"); // unchecked cast, fragile

// Good — deserialise into a typed record at the boundary
public record WeatherReport(
    String city,
    double temperature,
    double humidity,
    String conditions
) {}

public WeatherReport fetchWeather(String city) {
    var json = httpClient.get("https://api.weather.example/" + city);
    return objectMapper.readValue(json, WeatherReport.class);
}

var report = fetchWeather("London");
double temp = report.temperature(); // type-safe, IDE-navigable
```

---

## 6. Documentation & Comments

### 6.1 Javadoc

Every public class, interface, method, and field **must** have Javadoc:

```java
/**
 * Retrieve a user by their unique identifier.
 *
 * <p>Looks up the user in the cache first, then falls back to the database.
 *
 * @param userId the unique identifier for the user
 * @param includeDeleted if {@code true}, soft-deleted users are also returned
 * @return the matching user
 * @throws UserNotFoundException if no user matches the given id
 */
public User findUser(long userId, boolean includeDeleted) {
    // ...
}
```

- **First sentence** is the summary — imperative mood, ends with a period.
- Use `@param`, `@return`, `@throws`, `@see`, `@since`.
- Use `{@code ...}` for inline code and `{@link ClassName}` for references.
- Use `<p>` to separate paragraphs within Javadoc.

### 6.2 Comments (*Clean Code* Ch. 4)

- Comments explain **why**, not what.
- Never commit commented-out code.
- Use `// TODO:` and `// FIXME:` sparingly — track in the issue tracker.
- Use `@SuppressWarnings` with a comment explaining why the suppression is safe.

---

## 7. Error Handling

### 7.1 Principles (*Effective Java* Items 69-77, *Clean Code* Ch. 7)

- Use **checked exceptions** for recoverable conditions.
- Use **unchecked exceptions** (`RuntimeException`) for programming errors.
- **Never catch `Exception` or `Throwable`** without re-throwing or handling
  specifically.
- **Fail fast** — validate at boundaries.

### 7.2 Exception Hierarchy

```java
public class AppException extends Exception {
    public AppException(String message) { super(message); }
    public AppException(String message, Throwable cause) { super(message, cause); }
}

public class NotFoundException extends AppException {
    public NotFoundException(String resource, Object id) {
        super(String.format("%s %s not found", resource, id));
    }
}
```

### 7.3 Try-with-Resources (*Effective Java* Item 9)

**Always** use try-with-resources for `AutoCloseable` objects:

```java
try (var conn = dataSource.getConnection();
     var stmt = conn.prepareStatement(sql)) {
    stmt.setLong(1, userId);
    try (var rs = stmt.executeQuery()) {
        // process results
    }
}
```

- Never rely on `finalize()` — it is deprecated and unreliable.
- Use `Closeable` / `AutoCloseable` for custom resources.

### 7.4 Logging

- Use SLF4J + Logback (or Log4j2). Never `System.out.println()`.
- Log at appropriate levels: `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`.
- Use parameterised messages: `log.info("User {} logged in", userId)`.

---

## 8. Packages & Imports

### 8.1 Package Design

- Packages provide **one cohesive concept**: `com.example.auth`,
  `com.example.billing`.
- No `util`, `common`, `misc` — they hide unclear boundaries.
- Use package-private access as the default. Export only what clients need.

### 8.2 Import Ordering

Group imports separated by blank lines:

1. `java.*`
2. `javax.*`
3. Third-party (alphabetical)
4. Project packages

- **No wildcard imports** (`import java.util.*`). Import specific classes.
- Use `import static` sparingly — only for constants and assertion methods.

### 8.3 One Library per Concern

| Overlapping libraries | Pick one |
|---|---|
| `java.util.logging` / SLF4J / Log4j2 | SLF4J (facade) |
| `java.net.HttpURLConnection` / Apache HttpClient / OkHttp | One HTTP client |
| `Jackson` / `Gson` / `Moshi` | One JSON library |
| `JUnit 4` / `JUnit 5` | JUnit 5 |
| `Guava` / Apache Commons / stdlib | Stdlib first, then one utility library |
| `Lombok` / records / manual | Records for data carriers, Lombok sparingly |

---

## 9. Design Patterns in Java

### 9.1 Creational Patterns

#### Static Factory Methods (*Effective Java* Item 1)

```java
public final class Money {
    private final BigDecimal amount;
    private final Currency currency;

    private Money(BigDecimal amount, Currency currency) {
        this.amount = amount;
        this.currency = currency;
    }

    public static Money of(BigDecimal amount, Currency currency) {
        return new Money(amount, currency);
    }

    public static Money usd(double amount) {
        return new Money(BigDecimal.valueOf(amount), Currency.getInstance("USD"));
    }
}
```

#### Builder (*Effective Java* Item 2)

```java
public final class ServerConfig {
    private final String host;
    private final int port;
    private final boolean tls;

    private ServerConfig(Builder builder) {
        this.host = builder.host;
        this.port = builder.port;
        this.tls = builder.tls;
    }

    public static class Builder {
        private final String host;
        private int port = 8080;
        private boolean tls = false;

        public Builder(String host) {
            Objects.requireNonNull(host, "host");
            if (host.isBlank()) {
                throw new IllegalArgumentException("host must not be blank");
            }
            this.host = host;
        }

        public Builder port(int port) {
            if (port < 1 || port > 65535) {
                throw new IllegalArgumentException(
                    "port must be 1..65535, got " + port);
            }
            this.port = port;
            return this;
        }

        public Builder tls(boolean tls) { this.tls = tls; return this; }

        public ServerConfig build() {
            // Final cross-field validation goes here when applicable.
            return new ServerConfig(this);
        }
    }
}
```

#### Singleton

Prefer enum singletons (*Effective Java* Item 3):

```java
public enum Registry {
    INSTANCE;

    public void register(String name, Object service) { /* ... */ }
}
```

### 9.2 Structural Patterns

#### Decorator

```java
public class LoggingService implements UserService {
    private final UserService delegate;
    private final Logger log;

    public LoggingService(UserService delegate, Logger log) {
        this.delegate = delegate;
        this.log = log;
    }

    @Override
    public User findById(long id) {
        log.info("Finding user {}", id);
        return delegate.findById(id);
    }
}
```

#### Adapter

Implement a target interface by delegating to an incompatible type.

### 9.3 Behavioural Patterns

#### Strategy

```java
public interface Compressor {
    byte[] compress(byte[] data);
}

public class Archiver {
    private final Compressor compressor;

    public Archiver(Compressor compressor) {
        this.compressor = compressor;
    }

    public byte[] archive(byte[] payload) {
        return compressor.compress(payload);
    }
}
```

#### Observer

Use Java's functional interfaces or a lightweight event bus.

#### Template Method

```java
public abstract class DataPipeline {
    public final void run(String source) {
        byte[] raw = extract(source);
        Map<String, Object> data = transform(raw);
        load(data);
    }

    protected abstract byte[] extract(String source);
    protected abstract Map<String, Object> transform(byte[] raw);
    protected abstract void load(Map<String, Object> data);
}
```

---

## 10. Testing

### 10.1 Principles

- Use **JUnit 5** (Jupiter). Tests are first-class code.
- **Test behaviour, not implementation.** Tests should survive refactors.
- Follow the **testing pyramid**: unit > integration > e2e.

### 10.2 Naming

```java
@Test
void withdraw_insufficientFunds_throwsException() { /* ... */ }

@Test
void parseCsv_emptyFile_returnsEmptyList() { /* ... */ }
```

### 10.3 Structure (Arrange-Act-Assert)

```java
@Test
void applyDiscount_reducesTotal() {
    // Arrange
    var cart = new ShoppingCart(List.of(new Item("Book", 20.00)));
    var discount = new PercentageDiscount(10);

    // Act
    cart.applyDiscount(discount);

    // Assert
    assertEquals(18.00, cart.getTotal(), 0.001);
}
```

### 10.4 Assertions

- Use **AssertJ** for fluent, readable assertions:
  `assertThat(result).isNotNull().hasSize(3)`.
- Use `assertThrows()` for exception testing.

### 10.5 Mocking

- Use **Mockito**. Mock only at **system boundaries** (databases, HTTP, clock).
- Prefer **dependency injection** over mocking.
- Never mock the class under test.

---

## 11. Database Access & ACID

### 11.1 ACID at a Glance

| Property | Guarantee | Violation Example |
|---|---|---|
| **Atomicity** | A transaction completes entirely or has no effect. | Debit succeeds but credit crashes. |
| **Consistency** | All constraints hold after the transaction. | FK violation not caught. |
| **Isolation** | Concurrent transactions don't interfere. | Double withdrawal. |
| **Durability** | Committed data survives crashes. | Data lost after restart. |

### 11.2 Always Use Explicit Transactions

```java
try (var conn = dataSource.getConnection()) {
    conn.setAutoCommit(false);
    try (var debit = conn.prepareStatement(
            "UPDATE accounts SET balance = balance - ? WHERE id = ?");
         var credit = conn.prepareStatement(
            "UPDATE accounts SET balance = balance + ? WHERE id = ?")) {

        debit.setBigDecimal(1, amount);
        debit.setLong(2, fromId);
        debit.executeUpdate();

        credit.setBigDecimal(1, amount);
        credit.setLong(2, toId);
        credit.executeUpdate();

        conn.commit();
    } catch (SQLException e) {
        conn.rollback();
        throw e;
    }
}
```

### 11.3 Use Parameterised Queries

```java
// Yes — PreparedStatement
var stmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
stmt.setLong(1, userId);

// No — string concatenation (SQL injection)
var stmt = conn.createStatement();
stmt.executeQuery("SELECT * FROM users WHERE id = " + userId);
```

### 11.4 Connection Lifecycle

- Use **connection pools** (HikariCP).
- Always close connections, statements, and result sets via try-with-resources.
- Configure pool: `maximumPoolSize`, `minimumIdle`, `connectionTimeout`.

### 11.5 ORM (JPA/Hibernate)

- Use `@Transactional` with Spring, or `EntityManager.getTransaction()`.
- Prefer JPQL or Criteria API — never string-concatenated SQL.
- Use `@Version` for optimistic locking.
- Keep transactions short.

Spring `@Transactional` defaults that are easy to get wrong:

- **Propagation** defaults to `REQUIRED` — joins the caller's transaction
  if one exists, otherwise starts a new one.
- **Isolation** defaults to `DEFAULT` — uses the underlying datasource's
  isolation level (typically `READ_COMMITTED` on PostgreSQL/Oracle/SQL
  Server, `REPEATABLE_READ` on MySQL/InnoDB). Set it explicitly when the
  default isolation is wrong for your use case.
- **Rollback rules** default to rolling back on unchecked exceptions
  (`RuntimeException` / `Error`) only. Checked exceptions do **not**
  trigger rollback unless declared with `rollbackFor = ...`.
- Self-invocation (`this.someTransactionalMethod()` from within the same
  bean) bypasses the proxy and **does not** start a transaction.

### 11.6 SQL Injection Protection

SQL injection remains one of the most dangerous and common vulnerabilities.
**Never** build SQL queries with string concatenation or `String.format`.

- **Always use `PreparedStatement`** with `?` placeholders. Never use
  `Statement` with concatenation.
- Use **JPA/Hibernate named parameters** (`:param`) or positional parameters.
- Use **`CriteriaBuilder`** for dynamic queries instead of building SQL
  strings.
- Validate and constrain input before it reaches the database layer.

```java
// Bad — string concatenation (SQL injection vector)
var stmt = conn.createStatement();
stmt.executeQuery("SELECT * FROM users WHERE name = '" + name + "'");

// Bad — String.format (still injectable)
var sql = String.format("SELECT * FROM users WHERE name = '%s'", name);

// Good — PreparedStatement with placeholders
var stmt = conn.prepareStatement("SELECT * FROM users WHERE name = ?");
stmt.setString(1, name);

// Good — JPA named parameter
@Query("SELECT u FROM User u WHERE u.name = :name")
List<User> findByName(@Param("name") String name);

// Good — CriteriaBuilder for dynamic queries
var cb = em.getCriteriaBuilder();
var cq = cb.createQuery(User.class);
var root = cq.from(User.class);
cq.where(cb.equal(root.get("name"), name));
```

---

## 12. Concurrency

### 12.1 Principles (*Effective Java* Items 78-84)

- **Prefer immutability.** Immutable objects are inherently thread-safe.
- **Synchronise access to shared mutable state** or use `java.util.concurrent`.
- Use **higher-level concurrency utilities** (`ExecutorService`,
  `CompletableFuture`, `ConcurrentHashMap`) over raw `synchronized`/`wait`/`notify`.

### 12.2 Virtual Threads (Java 21+)

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    var futures = urls.stream()
        .map(url -> executor.submit(() -> fetch(url)))
        .toList();
    for (var future : futures) {
        process(future.get());
    }
}
```

- Virtual threads are cheap — create millions, don't pool them.
- In Java 21, prefer `ReentrantLock` over `synchronized` in code that may run
  on a virtual thread: `synchronized` pins the carrier thread for the entire
  critical section. JDK 24 removed this pinning, so the recommendation is
  much weaker on newer JDKs — but `ReentrantLock` remains preferable when
  you also need `tryLock`, fairness, or interruptible locking.
- `Executors.newVirtualThreadPerTaskExecutor()` returns an `ExecutorService`
  that `close()` (called by try-with-resources) will wait on — it blocks
  until all submitted tasks complete.

### 12.3 CompletableFuture

```java
CompletableFuture.supplyAsync(() -> fetchUser(id))
    .thenApply(this::enrichUser)
    .thenAccept(this::saveUser)
    .exceptionally(ex -> { log.error("Failed", ex); return null; });
```

### 12.4 Thread Safety

- Use `ConcurrentHashMap` instead of `Collections.synchronizedMap()`.
- Use `AtomicInteger`, `AtomicReference` for lock-free updates.
- Use `volatile` only for visibility, not for atomicity.
- Never call `Thread.stop()` or `Thread.suspend()`.

---

## 13. Performance & Idiomatic Java

### 13.1 Streams (Java 8+)

```java
var activeNames = users.stream()
    .filter(User::isActive)
    .map(User::getName)
    .sorted()
    .toList();
```

- Use streams for data transformations. Use loops for side-effectful code.
- Avoid parallel streams unless measured bottleneck exists.

### 13.2 Optional (*Effective Java* Item 55)

- Return `Optional<T>` instead of `null` from methods.
- Never use `Optional` as a field type or method parameter.
- Never call `get()` without `isPresent()` — use `orElse()`, `orElseThrow()`,
  `map()`, `flatMap()`.

### 13.3 String Handling

- Use `StringBuilder` for repeated concatenation in loops.
- Use text blocks (Java 15+) for multi-line strings.
- Use `String.formatted()` or `String.format()` for templating.

### 13.4 Collections

- Use `List.of()`, `Set.of()`, `Map.of()` for unmodifiable collections (Java 9+).
- Pre-size collections: `new ArrayList<>(expectedSize)`.
- Use `EnumMap` and `EnumSet` for enum-keyed maps and sets.

### 13.5 Pattern Matching (Java 21+)

```java
return switch (shape) {
    case Circle c -> Math.PI * c.radius() * c.radius();
    case Rectangle r -> r.width() * r.height();
    case Triangle t -> 0.5 * t.base_() * t.height();
};
```

Because `Shape` is `sealed` and the `switch` covers every permitted subtype,
the compiler treats this `switch` as exhaustive — no `default` branch is
needed (or wanted, since adding a new permitted subtype should cause a
compile-time error here).

### 13.6 Guard Clauses

```java
public Receipt processOrder(Order order) {
    if (order.isCancelled()) {
        throw new OrderCancelledException(order.getId());
    }
    if (order.getItems().isEmpty()) {
        return Receipt.empty();
    }
    return buildReceipt(order);
}
```

---

## 14. Defensive Programming & Input Validation

### 14.1 Validate at Boundaries

All external input — controller parameters, API payloads, file contents, CLI
arguments, environment variables — must be validated at the controller or
service boundary before it flows deeper into the application.

- Use **Bean Validation annotations** for declarative validation:
  `@NotNull`, `@NotBlank`, `@Size(min = 1, max = 255)`, `@Min`, `@Max`,
  `@Pattern`.
- Validate method arguments with `Objects.requireNonNull()` or Guava
  `Preconditions`.

```java
public record CreateUserRequest(
    @NotBlank @Size(max = 100) String name,
    @NotBlank @Email String email,
    @Min(0) @Max(150) int age
) {}

public User createUser(CreateUserRequest request) {
    Objects.requireNonNull(request, "request must not be null");
    // Bean Validation triggers automatically via @Valid in controllers
    return userRepository.save(new User(request.name(), request.email(), request.age()));
}
```

### 14.2 String and Numeric Validation

When annotations are not applicable, validate explicitly:

```java
public void updateUsername(User user, String newName) {
    Objects.requireNonNull(newName, "newName must not be null");
    if (newName.isBlank()) {
        throw new IllegalArgumentException("username cannot be blank");
    }
    if (newName.length() > 50) {
        throw new IllegalArgumentException("username too long (max 50)");
    }
    user.setUsername(newName);
}

public void setQuantity(Item item, int qty) {
    if (qty < 1 || qty > 9999) {
        throw new IllegalArgumentException("quantity must be 1..9999, got " + qty);
    }
    item.setQuantity(qty);
}
```

### 14.3 Domain Constraints

- Use **enums** and **sealed types** to constrain domains and eliminate
  invalid states at compile time.
- Use **records** for validated value objects with compact constructors:

```java
public record Port(int value) {
    public Port {
        if (value < 1 || value > 65535) {
            throw new IllegalArgumentException("Port must be 1..65535, got " + value);
        }
    }
}
```

### 14.4 Collection Safety

- Check collection bounds before indexing.
- Use `Optional` to avoid null dereferences.
- Use `List.of()`, `Map.of()` for unmodifiable collections that prevent
  accidental mutation.

### 14.5 Path and Process Safety

- Sanitize input used in file paths with `Path.normalize()` and
  `Path.toRealPath()`. Reject paths that escape the expected directory.
- **Never** pass user input directly to `Runtime.exec()` or
  `ProcessBuilder` without sanitization.
- **Never** use `Class.forName()` or reflection with unsanitized user input.

```java
// Bad — path traversal
Path file = Path.of("uploads", userInput);

// Good — normalise and constrain
Path base = Path.of("uploads").toAbsolutePath().normalize();
Path file = base.resolve(userInput).normalize();
if (!file.startsWith(base)) {
    throw new SecurityException("path traversal detected");
}
```

---

## 15. Project Structure

### 15.1 Recommended Layout (Maven/Gradle)

```
my-app/
    pom.xml / build.gradle
    src/
        main/
            java/
                com/example/myapp/
                    Application.java
                    config/
                    model/
                    service/
                    repository/
                    controller/
            resources/
                application.yml
        test/
            java/
                com/example/myapp/
                    service/
                        UserServiceTest.java
            resources/
```

### 15.2 Module System (Java 9+)

Use `module-info.java` for strong encapsulation:

```java
module com.example.myapp {
    requires java.sql;
    requires com.google.gson;       // automatic module name for gson.jar
    exports com.example.myapp.api;
}
```

Notes:

- `requires` references **module names**, not Maven/Gradle artifact coordinates.
  Use the published module name or the JAR's automatic module name (derived
  from `Automatic-Module-Name` in `MANIFEST.MF`, or from the JAR file name).
- `requires transitive` propagates a dependency to consumers of this module.
- `requires static` marks a compile-time-only dependency.

### 15.3 Build Systems

Java projects typically use **Maven** or **Gradle** for reproducible builds and dependency management.

#### Maven

Maven uses a declarative `pom.xml` to define project structure, dependencies, and build phases:

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
    
    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.10.0</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>21</source>
                    <target>21</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

Key Maven concepts:
- **Lifecycle**: `clean`, `compile`, `test`, `package`, `install`, `deploy`
- **Scopes**: `compile` (default), `provided`, `runtime`, `test`, `system`
- **Repositories**: Central, corporate nexus, artifact hosting

#### Gradle

Gradle uses imperative scripts for flexible build configuration:

```gradle
plugins {
    id 'java'
}

java {
    sourceCompatibility = '21'
}

dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.10.0'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

Key Gradle concepts:
- **Configurations**: `implementation`, `compileOnly`, `runtimeOnly`, `testImplementation`
- **Tasks**: Build tasks can be custom or from plugins
- **Kotlin DSL**: Modern Gradle uses `build.gradle.kts` (Kotlin) instead of Groovy

#### Ant (Legacy)

Ant is rarely used in modern Java projects but remains in legacy codebases. Use Maven or Gradle for new projects.

### 15.4 Dependency Management

- Use **Maven** or **Gradle**. Pin dependency versions.
- Separate compile, runtime, and test dependencies.
- Use BOM (Bill of Materials) for framework version alignment.
- Run `mvn dependency:analyze` / `gradle dependencies` regularly.

---

## 16. Tooling

### 16.1 Recommended Tool Chain

| Purpose | Tool | Notes |
|---|---|---|
| Formatter | **google-java-format** | Enforced in CI |
| Linter | **Error Prone**, **SpotBugs** | Catch common bugs at compile time |
| Static analysis | **SonarQube** / **SonarLint** | Continuous inspection |
| Test runner | **JUnit 5** | With JaCoCo for coverage |
| Assertions | **AssertJ** | Fluent, readable assertions |
| Mocking | **Mockito** | System boundary mocking |
| Build | **Maven** or **Gradle** | Reproducible builds |
| Dependency audit | **OWASP Dependency-Check** | Vulnerability scanning |
| IDE | **IntelliJ IDEA** | Inspections, refactoring, formatting |

### 16.2 CI Checks

```bash
mvn verify                           # compile, test, package
mvn spotbugs:check                   # bug detection
mvn com.spotify.fmt:fmt-maven-plugin:check  # formatting
mvn org.owasp:dependency-check-maven:check  # vulnerability scan
```

---

## 17. SBOM Creation

### 17.1 What is an SBOM?

A Software Bill of Materials (SBOM) documents all dependencies, transitive libraries, and frameworks used in a project. Essential for security audits, license compliance, and supply chain visibility.

### 17.2 Maven SBOM Generation

**CycloneDX Maven Plugin** (recommended):

```bash
mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom
```

Or add to `pom.xml`:

```xml
<plugin>
    <groupId>org.cyclonedx</groupId>
    <artifactId>cyclonedx-maven-plugin</artifactId>
    <version>2.8.0</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>makeAggregateBom</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### 17.3 Gradle SBOM Generation

**CycloneDX Gradle Plugin**:

```gradle
plugins {
    id "org.cyclonedx.bom" version "1.8.2"
}

cyclonedxBom {
    includeConfigs = ["runtimeClasspath"]
    skipConfigs = ["compileClasspath", "testCompileClasspath"]
    outputFormat = "json"
    outputName = "sbom"
}
```

Run: `gradle cyclonedxBom`

### 17.4 Vulnerability Scanning

**OWASP Dependency-Check** (Maven):

```bash
mvn org.owasp:dependency-check-maven:check
```

**Gradle**:

```gradle
plugins {
    id "org.owasp.dependencycheck" version "10.0.4"
}
```

### 17.5 License Compliance

Use `license-maven-plugin` (from `org.codehaus.mojo`) or Gradle equivalents:

```bash
mvn license:aggregate-third-party-report  # Lists all dependency licenses
```

### 17.6 Integration into CI/CD

- Generate SBOM in CI/CD on every release
- Scan for known vulnerabilities: `dependency-check`
- Verify license compliance
- Store SBOM artifacts (JSON/XML) alongside release binaries

---

## 18. References

### Official Documentation

| Resource | URL |
|---|---|
| Java Language Specification | https://docs.oracle.com/javase/specs/ |
| Google Java Style Guide | https://google.github.io/styleguide/javaguide.html |
| Java API Documentation | https://docs.oracle.com/en/java/javase/ |
| JUnit 5 User Guide | https://junit.org/junit5/docs/current/user-guide/ |

### Books

| Book | Authors | Key Takeaways for This Guide |
|---|---|---|
| *Effective Java* | Joshua Bloch (2001, 2008, 2018) | Static factories, builders, immutability, generics, lambdas, streams, concurrency — the definitive Java best practices. |
| *Design Patterns: Elements of Reusable Object-Oriented Software* | Gamma, Helm, Johnson, Vlissides (1994) | Composition over inheritance, program to interfaces. |
| *Clean Code: A Handbook of Agile Software Craftsmanship* | Robert C. Martin (2008) | Small methods, meaningful names, SRP, error handling. |
| *Java Concurrency in Practice* | Goetz et al. (2006) | Thread safety, visibility, atomicity, concurrent collections, executor framework. |
| *Modern Java in Action* | Urma, Fusco, Mycroft (2018) | Lambdas, streams, Optional, CompletableFuture, modules. |
| *The Pragmatic Programmer* | Hunt & Thomas (1999, 2019) | DRY, orthogonality, tracer bullets — universal engineering wisdom. |
