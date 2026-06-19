# Swift — AI Coding Rules

Apply these rules when generating or reviewing Swift code.

## Formatting

- 4 spaces indentation. Never tabs.
- 100 characters recommended line limit.
- Opening brace on same line.
- One blank line between method definitions.
- No blank line after opening brace or before closing brace.
- `// MARK: -` to organise sections within a file.
- Use SwiftFormat or swift-format for enforcement.

## Naming (Swift API Design Guidelines)

- `PascalCase` for types (struct, class, enum, protocol).
- `camelCase` for methods, properties, variables, enum cases.
- `camelCase` for constants (`let`), including static constants.
- Boolean: reads as assertion (`isEmpty`, `hasPermission`, `canEdit`).
- Protocol (capability): `-able`, `-ible`, `-ing` suffix (`Equatable`, `Codable`).
- Protocol (role): noun (`Collection`, `View`).
- Clarity at the point of use. Omit needless words.
- Name variables by role, not type: `greeting` not `greetingString`.
- File name matches primary type.

## Functions & Methods

- Small (under 20 lines), one thing, one level of abstraction.
- Use argument labels for clarity at call site: `move(_ piece: Piece, to destination: Square)`.
- Omit first argument label when method name makes it clear: `contains(_:)`.
- Use default values instead of overloads.
- Trailing closure syntax when closure is last argument.
- `[weak self]` or `[unowned self]` to break retain cycles.

## Types & Composition

- Prefer structs (value types) by default. Classes only when identity matters.
- Use enums with associated values to make illegal states unrepresentable.
- Exhaustive `switch` — no `default` unless genuinely open-ended.
- Protocol-oriented programming: prefer protocols over base classes.
- Protocol extensions for default implementations.
- `some Protocol` (opaque) and `any Protocol` (existential) appropriately.
- Use extensions to organise conformances.

## Typed Data Passing

- Pass typed values (structs, enums with associated values), not `[String: Any]` dictionaries, between functions and modules.
- Define clear types for all data crossing module boundaries.
- At system boundaries (API responses, config files, CLI args), decode into typed `Codable` structs immediately.
- Never use `Any` or `AnyObject` for structured data — use the type system and optionals.

## SOLID

- **S**: One type per concern.
- **O**: Extend via new protocol conformances and extensions.
- **L**: Subtypes must honour the protocol contract.
- **I**: Keep protocols small and focused.
- **D**: Accept protocols, not concrete types. Inject via init.

## Documentation (Swift Markup)

- `///` for doc comments on all `public` and `open` declarations.
- `- Parameter name:` for single, `- Parameters:` for multiple.
- `- Returns:`, `- Throws:`, `- Note:`, `- Precondition:`.

## Error Handling

- Typed errors: enum conforming to `Error` and `LocalizedError`.
- Use `throw`/`try`/`catch`. Propagate with `throws`.
- `try?` for nil-on-failure. `try!` only for guaranteed success.
- `Result<T, E>` for closures and async boundaries.
- `defer` for resource cleanup.

## Imports

- Order: system frameworks, third-party packages, internal modules.
- One library per concern: one networking, one persistence, one reactive/async framework.
- `@testable import` only in test targets.

## Concurrency (Swift 5.5+)

- `async`/`await` for all asynchronous code.
- `TaskGroup` for concurrent work. `Task` for fire-and-forget.
- `@MainActor` for UI-bound code.
- `actor` for thread-safe mutable state.
- Conform types to `Sendable` for cross-concurrency data; use `@Sendable` on closures crossing those boundaries. Under Swift 6 strict concurrency, all shared mutable state must be actor-isolated.

## Database / ACID

- Explicit transactions for multi-statement writes.
- Parameterised queries only — never string interpolation.
- Database work on background queues.
- Never use string interpolation (`"\(value)"`) to build SQL. Always use parameterised queries with bound parameters.
- Use library-specific parameterisation (GRDB, SQLite.swift, Core Data predicates).
- Validate and constrain input before it reaches the database layer.

## Testing

- XCTest or Swift Testing (Swift 6+).
- `testMethodName_Scenario_ExpectedResult` naming.
- Arrange-Act-Assert structure.
- Protocol-based mocking. Mock at system boundaries.

## Performance & Idioms

- Prefer `let` over `var`. Structs use copy-on-write.
- `guard let`/`if let` for unwrapping. Never force-unwrap (`!`) in production.
- `??` for defaults. `map`/`flatMap` for optional chaining.
- Functional: `.filter`, `.map`, `.sorted` with key paths.
- Guard clauses for early returns.

## Patterns

- Factory: static methods. Builder: method chaining or `@resultBuilder`.
- Singleton: `static let shared` — prefer DI instead.
- Strategy: protocol + DI. Observer: Combine, delegates, NotificationCenter.

## Defensive Programming & Input Validation

- Validate all external input at service/controller boundaries.
- Use `guard` statements for early validation and exit.
- Validate string lengths and numeric ranges before processing.
- Use enums to constrain value domains — make illegal states unrepresentable.
- Use `Optional` to represent absence — never sentinel values.
- Never force-unwrap (`!`) user-provided data.
- Sanitize input used in file paths, URLs (`URLComponents`), or process execution.
- Use `Codable` with custom `init(from:)` for validated deserialization.
- Use `@available` and `#available` for API availability checks.

## Project Structure

- Swift Package Manager preferred. `Sources/`, `Tests/` layout.
- Xcode: `Models/`, `Views/`, `ViewModels/`, `Services/`.

## Tooling

- SwiftFormat / swift-format (formatter), SwiftLint (linter).
- XCTest / Swift Testing (test), DocC (documentation).

## Build Tools

- Swift Package Manager (SPM): native, no external tool needed.
- `swift build`, `swift test`, `swift run` cover most needs.
- Xcode for iOS/macOS apps: `xcodebuild` for CLI builds.
- SwiftLint for code style enforcement.
- Support multiple platforms in Package.swift.
- Docker builds for server-side Swift applications.

## SBOM Creation

- Include `Package.resolved` in VCS for reproducible builds.
- Use `swift package show-dependencies --format json` for SBOM export.
- Run `snyk test` or GitHub Dependabot on every PR.
- Document package licenses in `LICENSES/` directory.
- Monitor for vulnerabilities with GitHub Dependabot or Snyk.
- Store SBOM with release artifacts.
