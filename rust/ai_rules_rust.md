# Rust — AI Coding Rules

Apply these rules when generating or reviewing Rust code.

## Formatting

- `rustfmt` is non-negotiable. Run `cargo fmt`.
- Default 100 character line limit.
- One blank line between top-level items and between methods in `impl` blocks.
- No blank line at start/end of blocks.
- One blank line between `use` groups (std, external, crate).

## Naming (RFC 430)

- `snake_case`: functions, methods, variables, modules, crates.
- `PascalCase`: types (struct, enum, trait), enum variants.
- `UPPER_SNAKE_CASE`: constants, statics.
- Acronyms as words in PascalCase: `HttpClient`, not `HTTPClient`.
- Conversions: `as_` (cheap borrow), `to_` (expensive owned), `into_` (consumes self).
- Booleans: `is_empty()`, `has_key()`, `can_read()`.
- Constructor: `new()`. Fallible: `fn new(...) -> Result<Self, E>`.
- No stuttering: `http::Client`, not `http::HttpClient`.

## Functions

- Small, one thing. Max ~40 lines.
- Accept most general type: `&str` over `&String`, `&[T]` over `&Vec<T>`, `impl AsRef<Path>`.
- Use `Result<T, E>` for fallible ops, `Option<T>` for absence. Never sentinel values.
- Closures for short local logic (iterators). `fn` items for named reusable behaviour.

## Types & Traits

- No classes/inheritance. Compose via structs, enums, traits, generics.
- Enums for sum types — make illegal states unrepresentable.
- Newtype pattern for type safety: `struct UserId(u64)`.
- Derive: `Debug` always, `Clone`, `Copy`, `Default`, `PartialEq`, `Eq`, `Hash` as appropriate. `From`/`Into` are implemented (not derived without `derive_more`).
- Small, focused traits (1-2 methods). Use trait bounds, not concrete types.
- Trait objects (`dyn Trait`) for runtime polymorphism when generics cause bloat.
- Pass typed structs and enums between functions — not `HashMap<String, String>` or raw strings.
- Use `From`/`Into` traits for type conversions at boundaries.
- At system boundaries, deserialize into typed structs (via `serde`) immediately.
- Use newtypes to distinguish semantically different values of the same primitive type.

## Documentation (rustdoc)

- `///` for items, `//!` for module/crate. Markdown format.
- Every public item gets a doc comment.
- Required sections: summary, `# Examples` (runnable), `# Errors`, `# Panics`, `# Safety` (for `unsafe fn`).
- `//` comments explain why. `// SAFETY:` required above every `unsafe` block.

## Error Handling

- `Result<T, E>` for recoverable, `panic!` for programmer errors.
- Never `panic` in library code. Return `Result`.
- Define crate-level error enum. Implement `Display`, `Error`, `From`.
- Use `thiserror` for libraries, `anyhow` for applications.
- `?` operator for propagation. Never `unwrap()` in production — use `expect("reason")` if truly unrecoverable.

## Ownership & Borrowing

- `&T` for read-only, `&mut T` for mutation, move for ownership transfer.
- `Arc<Mutex<T>>` for shared mutable state across threads.
- Elide lifetimes when possible. Name descriptively when explicit: `'input`, `'ctx`.
- `Clone`/`Copy` only when appropriate. Unnecessary `.clone()` is a code smell.

## Modules & Imports

- Group `use`: std, external, crate — separated by blank lines.
- Nested imports: `use std::io::{self, BufRead, Write}`.
- No glob imports (`*`) except in test modules.
- One crate per concern: one JSON lib, one HTTP client, one async runtime, one logger.
- Re-export key types at crate root for ergonomic public API.

## Concurrency

- `Send`/`Sync` for thread safety (compiler-verified).
- `Arc<Mutex<T>>` or `Arc<RwLock<T>>` for shared state. Keep lock scopes narrow.
- Channels for ownership transfer between threads.
- `async`/`await` with one runtime (tokio or async-std). Never block in async — use `spawn_blocking`.

## Unsafe

- **General prohibition:** Never use `unsafe` to work around borrow checker errors solvable with safe code.
- **Exceptions (strict limits):** FFI/ABI interfaces permitted when no safe alternative exists (C library bindings, system calls).
- `// SAFETY:` comment on every `unsafe` block. `# Safety` section on every `unsafe fn`.
- FFI unsafe requires: exhaustive C API documentation, invariant validity explanation, reference to official C API docs, encapsulation in safe wrapper, input validation at boundary.
- Test with `cargo +nightly miri test`.
- Use `cargo-deny` to enforce policies on unsafe usage.

## Testing

- `#[cfg(test)] mod tests` for unit tests. `tests/` for integration. Doc tests are real tests.
- `snake_case` naming: `withdraw_insufficient_funds_returns_error`.
- `assert_eq!` with descriptive messages. `matches!` for pattern assertions.

## Database

- Explicit transactions. `sqlx::Transaction` auto-rolls back on drop (RAII).
- Parameterised queries only — never `format!()` or string concatenation for SQL.
- Use `sqlx::query!()` macro for compile-time checked queries.
- Validate and constrain input before it reaches the database layer.
- Connection pools: `sqlx::PgPool`, `diesel::r2d2`.

## Patterns

- Constructors: `new()`, `with_*()`. Builder: owned builder consuming `self`.
- Singleton: `OnceLock` (1.70+). Decorator: trait wrapper.
- Newtype for type safety. Typestate for compile-time state machine validation.
- Strategy: trait or closure. Observer: `HashMap<String, Vec<Box<dyn Fn>>>`.

## Defensive Programming

- Validate all external input at system boundaries (HTTP handlers, CLI parsing, config loading).
- Use newtypes with validation in constructors: `impl Port { fn new(n: u16) -> Result<Self, Error> }`.
- Use `TryFrom`/`FromStr` for validated conversions. Use enums to make illegal states unrepresentable.
- Sanitize input in file paths (`Path::canonicalize`), URLs, or command execution.
- Never use `unsafe` to bypass validation. Use `#[must_use]` on Result types to prevent ignoring errors.
- The borrow checker and type system provide most defensive guarantees — leverage them.

## Project Structure

- `src/main.rs` or `src/lib.rs`. `tests/`, `benches/`, `examples/`.
- Keep `main()` thin: call `run() -> Result`. `Cargo.lock` committed for binaries.
- `[lints.rust] unsafe_code = "forbid"`. Clippy pedantic.

## Tooling

- `cargo fmt`, `cargo clippy -- -D warnings`, `cargo test`, `cargo audit`, `cargo doc --no-deps`.

## Build Tools

- Cargo is the standard and complete build system.
- `cargo build` (debug), `cargo build --release` (optimized).
- Define features in Cargo.toml for conditional compilation.
- Use workspaces for multi-crate projects.
- `build.rs` for compile-time code generation.
- `rustup target add` and `cross` for cross-platform builds.

## SBOM Creation

- Use `cargo-cyclonedx` for CycloneDX SBOM generation (JSON/XML).
- Run `cargo-audit` against RustSec advisory database.
- Use `cargo-license` for license compliance.
- Use `cargo-deny` for policy enforcement (licenses, advisories, unsafe patterns).
- Include `Cargo.lock` in VCS for binary crates; optional for libraries.
- Gate CI/CD on `cargo audit` and license checks.
