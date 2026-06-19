# Swift Coding Style Guidelines

A comprehensive guide rooted in the official Swift API Design Guidelines,
*The Swift Programming Language* (Apple), the Google Swift Style Guide, and
established software engineering literature — notably *Design Patterns*
(Gamma et al., 1994) and *Clean Code* (Martin, 2008).

---

## Table of Contents

1. [Philosophy](#1-philosophy)
2. [Code Layout & Formatting](#2-code-layout--formatting)
3. [Naming Conventions](#3-naming-conventions)
4. [Functions & Methods](#4-functions--methods)
5. [Types, Protocols & Composition](#5-types-protocols--composition)
6. [Documentation & Comments](#6-documentation--comments)
7. [Error Handling](#7-error-handling)
8. [Modules & Imports](#8-modules--imports)
9. [Design Patterns in Swift](#9-design-patterns-in-swift)
10. [Testing](#10-testing)
11. [Database Access & ACID](#11-database-access--acid)
12. [Concurrency](#12-concurrency)
13. [Performance & Idiomatic Swift](#13-performance--idiomatic-swift)
14. [Defensive Programming & Input Validation](#14-defensive-programming--input-validation)
15. [Project Structure](#15-project-structure)
16. [Tooling](#16-tooling)
17. [Build Tools](#17-build-tools)
18. [SBOM Creation](#18-sbom-creation)
19. [References](#19-references)

---

## 1. Philosophy

### 1.1 Swift's Core Values

Swift prioritises **safety**, **performance**, and **expressiveness**. The
type system, optionals, and value semantics are designed to eliminate entire
categories of bugs at compile time.

### 1.2 Guiding Principles

| Principle | Source | Summary |
|---|---|---|
| **Clarity at the point of use** | Swift API Design Guidelines | Names should read naturally at the call site. |
| **Value semantics by default** | Swift language design | Prefer structs and enums over classes. |
| **Protocol-oriented programming** | WWDC 2015, Swift community | Compose behaviour through protocols and extensions. |
| **Safety through optionals** | *The Swift Programming Language* | Use `Optional` instead of `nil` sentinels. |
| **Composition over inheritance** | *Design Patterns* Ch. 1 | Prefer protocols, extensions, and dependency injection. |
| **Single Responsibility** | *Clean Code* Ch. 10, SOLID | Every type and function should have one reason to change. |
| **YAGNI** | XP / *Clean Code* | Do not build for hypothetical future requirements. |

---

## 2. Code Layout & Formatting

### 2.1 Formatter

Use **swift-format** (Apple) or **SwiftFormat** (Nick Lockwood). Enforce in CI.

### 2.2 Indentation

- **4 spaces** per indentation level. Never tabs.

### 2.3 Line Length

- **100 characters** recommended maximum.

### 2.4 Blank Lines

- **One blank line** between method definitions.
- **One blank line** between property groups and method groups.
- **No blank line** after an opening brace or before a closing brace.
- **One blank line** between `import` groups (system, third-party, project).
- Use single blank lines within functions to separate logical sections.
- **No trailing blank lines** at the end of a file.

### 2.5 Braces

Opening brace on the same line:

```swift
if condition {
    doSomething()
} else {
    doOther()
}

func process(_ items: [Item]) {
    // ...
}
```

### 2.6 File Organisation

1. Imports
2. Protocols
3. Structs / Classes / Enums
4. Extensions (grouped by protocol conformance)
5. Private helpers

Use `// MARK: -` to organise sections within a file:

```swift
// MARK: - Public API
// MARK: - Private Helpers
// MARK: - CustomStringConvertible
```

---

## 3. Naming Conventions

### 3.1 General Rules (Swift API Design Guidelines)

| Entity | Convention | Example |
|---|---|---|
| Type (struct, class, enum, protocol) | `PascalCase` | `UserService`, `Codable` |
| Enum case | `camelCase` | `.darkBlue`, `.networkError` |
| Method / Function | `camelCase` | `calculateTotal()` |
| Variable / Property | `camelCase` | `totalCount` |
| Constant (`let`) | `camelCase` | `maximumRetries` |
| Static constant | `camelCase` | `Notification.Name.userDidLogin` |
| Type parameter | Single uppercase or short `PascalCase` | `T`, `Element` |
| Protocol (capability) | `-able`, `-ible`, `-ing` suffix | `Equatable`, `Codable`, `Loading` |
| Protocol (type role) | Noun | `Collection`, `Sequence`, `View` |
| Boolean | Reads as assertion | `isEmpty`, `hasPermission`, `canEdit` |
| File | Matches primary type | `UserService.swift` |

### 3.2 Naming Guidance

- **Clarity over brevity.** `removeItem(at:)` beats `remove(_:)` when the
  parameter role is ambiguous.
- **Omit needless words.** `removeItem(at:)` not `removeItemAtIndex(at:)`.
- **Name variables by role**, not type: `greeting` not `greetingString`.
- **Factory methods**: `making` for value types, `init` for reference types.
- Acronyms follow uniform case: `utf8`, `urlSession`, `HTTPURLResponse`
  (Apple convention preserves `URL`, `HTTP` in system frameworks).

---

## 4. Functions & Methods

### 4.1 Size and Focus

- Small (under 20 lines), one thing, one level of abstraction.

### 4.2 Parameters

- Use **argument labels** for clarity at the call site:

```swift
func move(_ piece: Piece, to destination: Square) { /* ... */ }
// Call: move(knight, to: e4)
```

- Omit the first argument label when the method name makes it clear:
  `contains(_:)`, `append(_:)`.
- Use default values instead of overloads.

### 4.3 Return Values

- Use `Optional` (`T?`) for values that may be absent.
- Use `Result<T, E>` for operations that can fail with typed errors.
- Use tuples for multiple related return values.

### 4.4 Closures

- Use trailing closure syntax when the closure is the last argument.
- Use `$0`, `$1` for short closures. Named parameters for longer ones.
- Mark closures `@escaping` when they outlive the function call.
- Use `[weak self]` or `[unowned self]` to break retain cycles.

---

## 5. Types, Protocols & Composition

### 5.1 Value Types vs. Reference Types

- **Prefer structs** (value types) by default.
- Use **classes** (reference types) only when identity matters, inheritance is
  needed, or interop with Objective-C is required.
- Use **enums** for finite, fixed sets of cases.

### 5.2 Enums — Make Illegal States Unrepresentable

```swift
enum ConnectionState {
    case disconnected
    case connecting(attempt: Int)
    case connected(peer: Peer)
    case failed(Error)
}
```

- Use associated values instead of optional fields.
- Exhaustive `switch` — no `default` unless genuinely open-ended.

### 5.3 Protocols (Protocol-Oriented Programming)

```swift
protocol Compressor {
    func compress(_ data: Data) -> Data
}

struct GzipCompressor: Compressor {
    func compress(_ data: Data) -> Data { /* ... */ }
}

struct Archiver {
    private let compressor: Compressor

    func archive(_ payload: Data) -> Data {
        compressor.compress(payload)
    }
}
```

- Prefer protocols over base classes.
- Use protocol extensions for default implementations.
- Use `some Protocol` (opaque types) and `any Protocol` (existentials)
  appropriately (Swift 5.7+).

### 5.4 Extensions

Use extensions to organise conformances and group related functionality:

```swift
extension User: Codable {
    // Codable implementation
}

extension User: CustomStringConvertible {
    var description: String { name }
}
```

### 5.5 Generics

```swift
func first<C: Collection>(_ collection: C) -> C.Element? {
    collection.first
}
```

- Use `where` clauses for complex constraints.
- Prefer constrained generics over `Any`.

### 5.6 Typed Data Passing

Pass structured, typed data between components — never raw dictionaries or
untyped values:

- Pass typed values (structs, enums with associated values), not
  `[String: Any]` dictionaries or raw strings, between functions and modules.
- Define clear types for all data that crosses module boundaries.
- At system boundaries (API responses, config files, CLI args), decode into
  typed `Codable` structs immediately.
- Use the type system and optionals to encode data contracts — never `Any` or
  `AnyObject` for structured data.

```swift
// Bad — untyped dictionary
func process(_ data: [String: Any]) { /* ... */ }

// Good — typed struct
struct UserPayload: Codable {
    let name: String
    let email: String
    let age: Int?
}

func process(_ payload: UserPayload) { /* ... */ }
```

### 5.7 SOLID in Swift

| Principle | Swift Application |
|---|---|
| **S** — Single Responsibility | One type per concern. |
| **O** — Open/Closed | Extend via new protocol conformances and extensions. |
| **L** — Liskov Substitution | Subtypes must honour the protocol contract. |
| **I** — Interface Segregation | Keep protocols small and focused. |
| **D** — Dependency Inversion | Accept protocols, not concrete types. Inject via init. |

---

## 6. Documentation & Comments

### 6.1 Doc Comments (Swift Markup)

Use `///` for documentation comments (Markdown format):

```swift
/// Retrieve a user by their unique identifier.
///
/// Looks up the user in the cache first, then falls back to the database.
///
/// - Parameters:
///   - userId: The unique identifier for the user.
///   - includeDeleted: If `true`, soft-deleted users are also returned.
/// - Returns: The matching user.
/// - Throws: `UserError.notFound` if no user matches the given id.
func findUser(userId: Int, includeDeleted: Bool = false) throws -> User {
    // ...
}
```

- Use `- Parameter name:` for single params, `- Parameters:` for multiple.
- Use `- Returns:`, `- Throws:`, `- Note:`, `- Important:`, `- Precondition:`.
- Document all `public` and `open` declarations.

### 6.2 Comments

- Comments explain **why**, not what. No commented-out code.
- Use `// MARK: -`, `// TODO:`, `// FIXME:` for organisation and tracking.

---

## 7. Error Handling

### 7.1 Principles

Swift uses **typed error handling** with `throw`/`try`/`catch`:

```swift
enum UserError: Error, LocalizedError {
    case notFound(id: Int)
    case invalidEmail(String)

    var errorDescription: String? {
        switch self {
        case .notFound(let id): return "User \(id) not found"
        case .invalidEmail(let email): return "Invalid email: \(email)"
        }
    }
}
```

### 7.2 Error Handling Patterns

```swift
// Propagate
func loadUser(id: Int) throws -> User {
    let data = try fetchData(for: id)
    return try decode(data)
}

// Handle
do {
    let user = try loadUser(id: 42)
} catch UserError.notFound(let id) {
    log.warning("User \(id) not found")
} catch {
    log.error("Unexpected error: \(error)")
}
```

- Use `try?` when you want `nil` on failure.
- Use `try!` only for values that are guaranteed to succeed (e.g., compile-time
  constants).
- Use `Result<T, E>` when passing errors through closures or async boundaries.

### 7.3 Resource Cleanup

Use `defer` for cleanup:

```swift
func processFile(at path: String) throws {
    let handle = try FileHandle(forReadingFrom: URL(fileURLWithPath: path))
    defer { handle.closeFile() }
    // process...
}
```

---

## 8. Modules & Imports

### 8.1 Module Design

- One module (framework/package target) per logical concern.
- Use `internal` (default) access. Expose only `public`/`open` API.
- `@testable import` for testing internal members.

### 8.2 Import Ordering

1. System frameworks (`Foundation`, `UIKit`, `SwiftUI`)
2. Third-party packages
3. Internal modules

### 8.3 One Library per Concern

| Overlapping libraries | Pick one |
|---|---|
| `URLSession` / Alamofire / Moya | One networking library |
| `Codable` / SwiftyJSON / ObjectMapper | `Codable` (stdlib) |
| `Core Data` / Realm / GRDB / SQLite.swift | One persistence library |
| `Combine` / RxSwift / AsyncAlgorithms | One reactive/async framework |
| `XCTest` / Quick+Nimble | One test framework |

---

## 9. Design Patterns in Swift

### 9.1 Creational

- **Factory**: static methods or standalone functions.
- **Builder**: method chaining with `self` return, or `@resultBuilder`.
- **Singleton**: `static let shared = MyService()` — use sparingly, prefer DI.

### 9.2 Structural

- **Decorator**: protocol wrapper. **Adapter**: protocol conformance wrapper.
- **Facade**: simplified API over complex subsystem.

### 9.3 Behavioural

- **Strategy**: protocol + DI. **Observer**: `Combine`, `NotificationCenter`,
  or delegate pattern. **Template Method**: protocol with default extension
  implementations.

---

## 10. Testing

### 10.1 Principles

- Use **XCTest** or **Swift Testing** (Swift 6+). Tests are first-class code.
- **Test behaviour, not implementation.** Tests should survive refactors.
- Testing pyramid: unit > integration > UI.

### 10.2 Naming

```swift
func testWithdraw_insufficientFunds_throwsError() { /* ... */ }
```

### 10.3 Structure (Arrange-Act-Assert)

```swift
func testApplyDiscount_reducesTotal() {
    // Arrange
    let cart = ShoppingCart(items: [Item("Book", price: 20.0)])
    let discount = PercentageDiscount(10)

    // Act
    cart.apply(discount)

    // Assert
    XCTAssertEqual(cart.total, 18.0, accuracy: 0.01)
}
```

### 10.4 Mocking

- Use **protocol-based mocking**. Define protocols for dependencies and
  supply test doubles.
- Mock at system boundaries: network, persistence, clock.

---

## 11. Database Access & ACID

### 11.1 ACID Properties

Same principles as all languages: Atomicity, Consistency, Isolation, Durability.

### 11.2 Core Data / GRDB / SQLite.swift

- Use explicit transactions for multi-statement writes.
- Parameterised queries only — never string interpolation.
- Perform database work on background queues.
- Use `NSManagedObjectContext.perform {}` for Core Data thread safety.

### 11.3 SQL Injection Protection

Never build SQL queries with string interpolation or concatenation. Always
use parameterised queries with bound parameters:

```swift
// Bad — string interpolation (SQL injection vulnerability)
let query = "SELECT * FROM users WHERE email = '\(email)'"
try db.execute(query)

// Good — parameterised query (GRDB)
let users = try User.filter(Column("email") == email).fetchAll(db)

// Good — parameterised query (SQLite.swift)
let query = users.filter(emailCol == email)

// Good — parameterised query (raw SQL with GRDB)
let rows = try Row.fetchAll(db,
    sql: "SELECT * FROM users WHERE email = ?",
    arguments: [email])
```

- Always use parameterised queries with bound parameters.
- Never use string interpolation (`"\(value)"`) to build SQL.
- Use library-specific parameterisation (GRDB, SQLite.swift, Core Data
  predicates).
- Validate and constrain input before it reaches the database layer.

---

## 12. Concurrency

### 12.1 Structured Concurrency (Swift 5.5+)

```swift
func fetchAll(urls: [URL]) async throws -> [Data] {
    try await withThrowingTaskGroup(of: Data.self) { group in
        for url in urls {
            group.addTask { try await fetchData(from: url) }
        }
        var results: [Data] = []
        for try await data in group {
            results.append(data)
        }
        return results
    }
}
```

- Use `async`/`await` for all asynchronous code.
- Use `TaskGroup` for concurrent work. `Task` for fire-and-forget.
- Use `@MainActor` for UI-bound code.
- Use `actor` for thread-safe mutable state.
- Conform types to `Sendable` for safe data sharing across concurrency
  boundaries. Annotate closures `@Sendable` when they cross such boundaries.
- Under Swift 6 strict concurrency, all global mutable state must be
  isolated to an actor (or marked `nonisolated(unsafe)` with explicit
  justification). Prefer `@MainActor` or a dedicated `actor` over locks.

### 12.2 Actors

```swift
actor AccountStore {
    private var accounts: [Int: Account] = [:]

    func deposit(amount: Decimal, to id: Int) {
        guard var account = accounts[id] else { return }
        account.balance += amount
        accounts[id] = account
    }
}
```

`Account` is a struct here — mutating a struct stored in a dictionary
requires a read-modify-write because `dict[key]?.field += x` operates on a
temporary value, not the stored copy. If `Account` were a class, you would
also need to mark it `Sendable` (or use a `final` class with internal
locking) to satisfy actor isolation.

---

## 13. Performance & Idiomatic Swift

### 13.1 Value Semantics

- Structs use copy-on-write. Prefer `let` over `var`.
- Use `inout` sparingly and only when mutation is intentional.

### 13.2 Functional Operations

```swift
let activeNames = users
    .filter(\.isActive)
    .map(\.name)
    .sorted()
```

### 13.3 Guard Clauses

```swift
func processOrder(_ order: Order) throws -> Receipt {
    guard !order.isCancelled else {
        throw OrderError.cancelled(order.id)
    }
    guard !order.items.isEmpty else {
        return .empty
    }
    return buildReceipt(for: order)
}
```

### 13.4 Optional Handling

- Use `guard let`/`if let` for unwrapping. Never force-unwrap (`!`) in
  production code (except `IBOutlet`).
- Use `??` for defaults. Use `map`/`flatMap` for chaining.
- Use `Optional` binding in switch cases.

---

## 14. Defensive Programming & Input Validation

### 14.1 Validate External Input at Boundaries

Validate all external input at service and controller boundaries. Never
trust data from users, APIs, files, or the network.

### 14.2 Guard Statements for Early Validation

Use `guard` statements for early validation and exit:

```swift
func createUser(name: String, port: Int) throws -> User {
    guard name.count <= 255 else {
        throw ValidationError.tooLong(field: "name", max: 255)
    }
    guard (1...65535).contains(port) else {
        throw ValidationError.outOfRange(field: "port", range: "1–65535")
    }
    return User(name: name, port: port)
}
```

### 14.3 Use the Type System

- Use enums to constrain value domains — make illegal states
  unrepresentable.
- Use `Optional` to represent absence — never use sentinel values (e.g.,
  `-1` for "not found").
- Never force-unwrap (`!`) user-provided data.

### 14.4 Validated Deserialization

Use `Codable` with custom `init(from:)` for validated deserialization:

```swift
struct UserRequest: Codable {
    let name: String
    let age: Int

    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        let name = try container.decode(String.self, forKey: .name)
        guard name.count <= 255 else {
            throw DecodingError.dataCorruptedError(
                forKey: .name, in: container,
                debugDescription: "Name exceeds 255 characters")
        }
        self.name = name
        self.age = try container.decode(Int.self, forKey: .age)
    }
}
```

### 14.5 Path, URL, and Process Safety

- Sanitize input used in file paths — validate against path traversal
  (`..`).
- Use `URLComponents` to build URLs safely — never string concatenation.
- Never pass unsanitized input to `Process` / shell execution.

### 14.6 API Availability

Use `@available` and `#available` for API availability checks:

```swift
if #available(iOS 16, *) {
    // Use new API
} else {
    // Fallback
}
```

---

## 15. Project Structure

### 15.1 Swift Package Manager

```
MyPackage/
    Package.swift
    Sources/
        MyLib/
            Models/
            Services/
        MyApp/
            main.swift
    Tests/
        MyLibTests/
```

### 15.2 Xcode Project

Follow Apple conventions: `Models/`, `Views/`, `ViewModels/`,
`Services/`, `Networking/`, `Extensions/`.

---

## 16. Tooling

| Purpose | Tool | Notes |
|---|---|---|
| Formatter | **SwiftFormat** / **swift-format** | Enforced in CI |
| Linter | **SwiftLint** | Community rule set |
| Test runner | **XCTest** / **Swift Testing** | Integrated with Xcode |
| Package manager | **Swift Package Manager** | First-party |
| Documentation | **DocC** | Generates from doc comments |
| Static analysis | **Xcode Analyzer** | Memory leaks, logic errors |

---

## 17. Build Tools

### 17.1 Swift Package Manager (SPM)

Swift's native build and dependency manager:

```bash
swift build  # Debug build
swift build -c release  # Release build
swift test  # Run tests
swift run myapp  # Run executable
```

Note: `swift package generate-xcodeproj` was removed in Swift 5.6. Open
`Package.swift` directly in Xcode instead.

Package.swift:
```swift
let package = Package(
    name: "MyApp",
    platforms: [.macOS(.v12)],
    products: [.executable(name: "myapp", targets: ["MyApp"])],
    dependencies: [
        .package(url: "https://github.com/swift-nio/swift-nio.git", from: "2.0.0"),
    ],
    targets: [
        .executableTarget(name: "MyApp", dependencies: [.product(name: "NIO", package: "swift-nio")]),
        .testTarget(name: "MyAppTests", dependencies: ["MyApp"]),
    ]
)
```

### 17.2 Xcode Build System (for macOS/iOS)

For iOS/macOS apps, use Xcode:
```bash
xcodebuild -scheme MyApp -configuration Release build
xcodebuild -scheme MyApp test
xcodebuild -scheme MyApp -configuration Release archive -archivePath ./MyApp.xcarchive
```

### 17.3 SwiftLint Integration

Add to build phases in Xcode or as a build script:
```bash
if which swiftlint >/dev/null; then
  swiftlint
else
  echo "warning: SwiftLint not installed"
fi
```

Or run manually:
```bash
swiftlint lint
swiftlint autocorrect
```

### 17.4 Cross-platform Builds

SPM supports multiple platforms:
```swift
.macOS(.v12),
.iOS(.v14),
.tvOS(.v15),
.watchOS(.v8)
```

Build for specific platform:
```bash
swift build -Xswiftc -target -Xswiftc x86_64-apple-macosx10.15
```

### 17.5 Docker for Swift Applications

```dockerfile
FROM swift:6.0 AS builder
WORKDIR /app
COPY . .
RUN swift build -c release

FROM swift:6.0-slim
COPY --from=builder /app/.build/release/myapp /app/myapp
CMD ["/app/myapp"]
```

Use the matching `swift:N-slim` (or `swift:N-runtime`) image as the runtime
stage so the Swift runtime libraries are present. Bare `ubuntu:22.04` will
fail to launch a Swift binary without manually installing libswift.

### 17.6 Continuous Integration

GitHub Actions for Swift:
```yaml
name: Build & Test
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: swift-actions/setup-swift@v2
      - run: swift build
      - run: swift test
```

---

## 18. SBOM Creation

### 18.1 What is an SBOM?

A Software Bill of Materials documents all Swift package dependencies. Essential for vulnerability tracking, license compliance, and supply chain security.

### 18.2 Swift Package Manager Dependency Management

Swift Package Manager locks dependencies via `Package.resolved`:
- `Package.swift`: Declares direct dependencies
- `Package.resolved`: Locks exact versions (must be in VCS)

View dependency tree:
```bash
swift package describe  # Summary of dependencies
swift package show-dependencies  # Detailed tree
```

### 18.3 SBOM Generation

**Generate from SwiftPM**:
```bash
swift package show-dependencies --format json > deps.json
```

Convert the dependency graph to CycloneDX with a community plugin (e.g.
`cyclonedx-swift`) or a custom script. There is no first-party Swift SBOM
plugin; verify any third-party tool's provenance before adopting it.

### 18.4 Vulnerability Scanning

**Using `Snyk` CLI** (or GitHub Dependabot):
```bash
snyk test  # Check for known vulns in dependencies
snyk monitor  # Continuous monitoring
```

**GitHub Dependabot** (native):
- Scans `Package.resolved`
- Creates PRs for updates
- Dashboard on Security tab

Swift Security Tool (under development) for supply chain security.

### 18.5 License Compliance

Swift packages should declare licenses in `Package.swift`:
```swift
let package = Package(
    name: "MyPackage",
    products: [...],
    dependencies: [...],
    targets: [...],
    platforms: [
        .macOS(.v12),
        .iOS(.v14)
    ]
)
```

Document dependencies and their licenses in `LICENSES/` directory.

### 18.6 Integration into CI/CD

- Include `Package.resolved` in VCS (required for reproducibility)
- Run `snyk test` or GitHub Dependabot on every PR
- Generate SBOM on release using a CycloneDX tool over the SwiftPM
  dependency graph
- Verify license compliance for dependencies
- Monitor for new vulnerabilities
- Store SBOM as release artifact

---

## 19. References

### Official Documentation

| Resource | URL |
|---|---|
| The Swift Programming Language | https://docs.swift.org/swift-book/ |
| Swift API Design Guidelines | https://www.swift.org/documentation/api-design-guidelines/ |
| Swift Evolution Proposals | https://github.com/apple/swift-evolution |
| DocC Documentation | https://www.swift.org/documentation/docc/ |

### Books

| Book | Authors | Key Takeaways for This Guide |
|---|---|---|
| *The Swift Programming Language* | Apple | Definitive language reference — optionals, protocols, generics, concurrency. |
| *Advanced Swift* | Eidhof, Airspeed Velocity, Velocity (objc.io) | Deep dive into protocols, generics, memory management, performance. |
| *Design Patterns: Elements of Reusable Object-Oriented Software* | Gamma et al. (1994) | Composition over inheritance; program to interfaces — protocols in Swift. |
| *Clean Code* | Robert C. Martin (2008) | Small functions, meaningful names, SRP. |
| *The Pragmatic Programmer* | Hunt & Thomas (1999, 2019) | DRY, orthogonality — universal engineering wisdom. |
