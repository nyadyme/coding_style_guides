# Rust Coding Style Guidelines

A comprehensive guide rooted in *The Rust Programming Language* ("The Book"),
*The Rustonomicon*, the Rust API Guidelines, the standard library conventions,
and established software engineering literature — notably *Design Patterns*
(Gamma et al., 1994) and *Clean Code* (Martin, 2008).

---

## Table of Contents

1. [Philosophy](#1-philosophy)
2. [Code Layout & Formatting](#2-code-layout--formatting)
3. [Naming Conventions](#3-naming-conventions)
4. [Functions](#4-functions)
5. [Types, Traits & Composition](#5-types-traits--composition)
6. [Documentation & Comments](#6-documentation--comments)
7. [Error Handling](#7-error-handling)
8. [Modules, Crates & Imports](#8-modules-crates--imports)
9. [Ownership, Borrowing & Lifetimes](#9-ownership-borrowing--lifetimes)
10. [Concurrency](#10-concurrency)
11. [Design Patterns in Rust](#11-design-patterns-in-rust)
12. [Testing](#12-testing)
13. [Database Access & ACID](#13-database-access--acid)
14. [Unsafe Code](#14-unsafe-code)
15. [Performance & Idiomatic Rust](#15-performance--idiomatic-rust)
16. [Defensive Programming & Input Validation](#16-defensive-programming--input-validation)
17. [Project Structure & Cargo](#17-project-structure--cargo)
18. [Tooling](#18-tooling)
19. [Build Tools](#19-build-tools)
20. [SBOM Creation](#20-sbom-creation)
21. [References](#21-references)

---

## 1. Philosophy

### 1.1 Rust's Core Values

Rust's design is driven by three pillars — **safety**, **speed**, and
**concurrency** — without requiring a garbage collector. The language enforces
correctness at compile time wherever possible.

Key beliefs from the Rust community:

> If it compiles, it works (modulo logic errors).
> Make illegal states unrepresentable.
> Fearless concurrency.
> Zero-cost abstractions.
> Explicit is better than implicit.
> Pay only for what you use.

### 1.2 Guiding Principles

| Principle | Source | Summary |
|---|---|---|
| **Leverage the type system** | *The Book* Ch. 17, Rust API Guidelines | Encode invariants in types so the compiler catches violations. |
| **Ownership over garbage collection** | *The Book* Ch. 4 | Every value has a single owner; borrowing is explicit and checked at compile time. |
| **Composition over inheritance** | Rust language design, *Design Patterns* Ch. 1 | Rust has no inheritance. Compose behaviour through traits, generics, and enums. |
| **Make illegal states unrepresentable** | Rust community convention | Use enums, newtypes, and the type system to make invalid data unconstructable. |
| **Explicit error handling** | *The Book* Ch. 9 | Use `Result<T, E>` and `Option<T>` — no exceptions, no null. |
| **Single Responsibility** | *Clean Code* Ch. 10, SOLID | Every module, type, and function should have one reason to change. |
| **YAGNI** | XP / *Clean Code* | Do not build for hypothetical future requirements. |
| **Unsafe is an escape hatch, not a shortcut** | *The Rustonomicon* | Minimise `unsafe`, isolate it, document its invariants exhaustively. |

---

## 2. Code Layout & Formatting

### 2.1 rustfmt Is Non-Negotiable

All Rust code **must** be formatted with `rustfmt`. There are no style
debates — the formatter is the authority.

- Run `cargo fmt` before every commit.
- Configure project-wide settings in `rustfmt.toml` if the team agrees on
  deviations from defaults (e.g. `max_width = 100`).
- Do not `#[rustfmt::skip]` unless formatting genuinely harms readability
  (e.g. hand-aligned lookup tables).

### 2.2 Line Length

The `rustfmt` default is **100 characters**. Keep this unless the team
explicitly agrees on a different value. Document any override in
`rustfmt.toml`.

### 2.3 Blank Lines (Rust Style Guide, `rustfmt`)

`rustfmt` enforces most blank-line rules automatically:

- **One blank line** between top-level items: `fn`, `struct`, `enum`, `impl`,
  `trait`, `mod`, `const`, `static`, `type`.
- **One blank line** between methods inside an `impl` block.
- **No blank line** between a function signature and its body.
- **No blank line** at the start or end of a block (`{` ... `}`).
- **One blank line** between use-declaration groups (stdlib, external, crate).
- Use **single blank lines** within functions to separate logical sections.
  Never use two or more consecutive blank lines.
- **One blank line** after the module-level doc comment (`//!`) before the
  first `use` declaration.

### 2.4 File Organisation

Within a `.rs` file, order elements as follows:

1. Module-level doc comment (`//!`)
2. `use` declarations
3. Constants and statics (`const`, `static`)
4. Type definitions (`struct`, `enum`, `type`)
5. Trait definitions
6. `impl` blocks for types (inherent impls before trait impls)
7. Free functions
8. Tests module (`#[cfg(test)]`)

### 2.5 The `main` Function

Every binary crate must have a `main()` function. Structure it to keep logic
testable:

```rust
use std::process;

fn main() {
    if let Err(e) = run() {
        eprintln!("error: {e}");
        process::exit(1);
    }
}

/// Run the application, returning any top-level error.
fn run() -> Result<(), Box<dyn std::error::Error>> {
    let config = Config::from_env()?;
    let app = App::new(config)?;
    app.start()
}
```

- Keep `main()` thin — parse args, call `run()`, handle the exit code.
- Return `Result` from `run()` so all error paths are testable.

---

## 3. Naming Conventions

Rust naming conventions are codified in the Rust API Guidelines (RFC 430).

### 3.1 General Rules

| Entity | Convention | Example |
|---|---|---|
| Crate | `snake_case` | `my_crate`, `serde_json` |
| Module | `snake_case` | `config`, `data_loader` |
| Type (struct, enum, trait) | `PascalCase` | `HttpClient`, `ParseError` |
| Enum variant | `PascalCase` | `Color::DarkBlue` |
| Function / Method | `snake_case` | `calculate_total()` |
| Local variable | `snake_case` | `total_count` |
| Constant | `UPPER_SNAKE_CASE` | `MAX_RETRIES` |
| Static | `UPPER_SNAKE_CASE` | `DEFAULT_CONFIG` |
| Type parameter | Single uppercase or short `PascalCase` | `T`, `K`, `V`, `Idx` |
| Lifetime | Short, lowercase, `'a`-style | `'a`, `'ctx`, `'de` |
| Feature flag | `kebab-case` | `serde-support` |
| Macro | `snake_case!` | `vec!`, `println!` |

### 3.2 Naming Guidance (Rust API Guidelines, *Clean Code* Ch. 2)

- **Conversion methods** follow `as_`, `to_`, `into_`:
  - `as_str()` — cheap, borrowed view (no allocation).
  - `to_string()` — potentially expensive, returns owned data.
  - `into_inner()` — consumes `self`, returns the inner value.
- **Boolean methods** read as assertions: `is_empty()`, `has_key()`,
  `can_read()`.
- **Constructor** convention is `new()` or `with_capacity()`, `from_str()`.
- **Fallible constructors** return `Result`: `fn new(...) -> Result<Self, E>`.
- **Use intention-revealing names.** `elapsed_time_in_secs` beats `d`.
- **No stuttering.** A struct in module `http` should be `Client`, not
  `HttpClient` — it's accessed as `http::Client`.
- **No encodings or prefixes.** No Hungarian notation, no `m_` member prefixes.
  The type system provides that information.

### 3.3 Casing for Acronyms

Follow Rust convention — acronyms in `PascalCase` names are capitalised as
words:

```rust
// Yes
struct HttpClient;
enum JsonValue { ... }
fn parse_url(input: &str) -> Url { ... }

// No
struct HTTPClient;
enum JSONValue { ... }
```

---

## 4. Functions

### 4.1 Size and Focus (*Clean Code* Ch. 3)

- Functions should be **small** and do **one thing** at **one level of
  abstraction**.
- If a function spans more than ~40 lines, consider splitting it.
- Extract deeply nested blocks into well-named helpers.

### 4.2 Parameters

- **Fewer parameters are better.** Beyond three, group into a config struct.
- Accept the **most general type** that works:
  - `&str` over `&String`
  - `&[T]` over `&Vec<T>`
  - `impl AsRef<Path>` over `&Path`
  - `impl Iterator<Item = T>` over `&Vec<T>` when only iteration is needed

```rust
/// Read all lines from a file at the given path.
///
/// # Errors
///
/// Returns an error if the file cannot be opened or read.
pub fn read_lines(path: impl AsRef<Path>) -> io::Result<Vec<String>> {
    let file = File::open(path)?;
    BufReader::new(file).lines().collect()
}
```

### 4.3 Return Values

- Use `Result<T, E>` for fallible operations.
- Use `Option<T>` for values that may be absent — never sentinel values.
- Return **owned data** from constructors and data-producing functions.
- Return **references** when lending access to internal state.

### 4.4 Closures

- Prefer closures for short, local logic — especially with iterators.
- Use `fn` items for named, reusable behaviour.
- Annotate closure parameter types when they are not obvious from context.

---

## 5. Types, Traits & Composition

Rust has no classes and no inheritance. Behaviour is built through **structs**,
**enums**, **traits**, and **generics**.

### 5.1 Structs

- Use structs for grouping related data.
- Prefer **public fields** for plain data containers (DTOs). Use private fields
  with accessor methods when invariants must be enforced.
- Derive common traits automatically:

```rust
/// A geographic coordinate.
#[derive(Debug, Clone, Copy, PartialEq)]
pub struct Coordinate {
    pub latitude: f64,
    pub longitude: f64,
}
```

### 5.2 Enums — Make Illegal States Unrepresentable

Rust enums are **sum types** — one of the language's most powerful tools:

```rust
/// Represents the possible states of a network connection.
#[derive(Debug)]
pub enum ConnectionState {
    Disconnected,
    Connecting { attempt: u32 },
    Connected { peer: SocketAddr },
    Failed { reason: String },
}
```

- Use enums instead of boolean flags, string tags, or integer codes.
- Combine with `match` for exhaustive handling — the compiler catches missing
  arms.
- Use the **newtype pattern** to add type safety without runtime cost:

```rust
/// A user ID, distinct from other integer identifiers.
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct UserId(u64);

/// An order ID, distinct from UserId even though both wrap u64.
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct OrderId(u64);
```

### 5.3 Traits (*The Book* Ch. 10, Rust API Guidelines)

Traits are Rust's mechanism for polymorphism and shared behaviour — analogous
to interfaces:

```rust
/// A type that can be serialized to bytes.
pub trait Serialize {
    /// Serialize this value into a byte vector.
    ///
    /// # Errors
    ///
    /// Returns an error if serialization fails.
    fn serialize(&self) -> Result<Vec<u8>, SerializeError>;
}
```

- **Keep traits small and focused.** A trait with one or two methods is ideal.
- **Implement standard traits** where applicable: `Display`, `Debug`, `Clone`,
  `Default`, `From`, `Into`, `PartialEq`, `Eq`, `Hash`.
- Use **trait bounds** to express requirements on generic types:

```rust
/// Log any displayable value.
fn log_value(value: &impl Display) {
    println!("[LOG] {value}");
}
```

- Use **trait objects** (`dyn Trait`) for runtime polymorphism when generics
  would cause code bloat or when the concrete type is unknown at compile time.

### 5.4 Composition via Traits and Generics

Instead of inheritance, compose behaviour:

```rust
/// A compressor compresses raw data.
pub trait Compressor {
    /// Compress the input data.
    fn compress(&self, data: &[u8]) -> Vec<u8>;
}

/// An archiver that uses a pluggable compression strategy.
pub struct Archiver<C: Compressor> {
    compressor: C,
}

impl<C: Compressor> Archiver<C> {
    /// Create a new archiver with the given compressor.
    pub fn new(compressor: C) -> Self {
        Self { compressor }
    }

    /// Archive the payload by compressing it.
    pub fn archive(&self, payload: &[u8]) -> Vec<u8> {
        self.compressor.compress(payload)
    }
}
```

### 5.5 Typed Data Passing

Pass **typed structs and enums** between functions and modules — not
`HashMap<String, String>`, raw JSON strings, or unstructured data. Rust's type
system is one of its greatest strengths; use it to make data contracts explicit
and compiler-verified.

- **Pass typed structs and enums**, not `HashMap<String, String>` or raw
  strings. Define a struct for any data that crosses a function or module
  boundary:

```rust
/// Data required to register a new user.
#[derive(Debug)]
pub struct CreateUserRequest {
    pub email: String,
    pub display_name: String,
}

// Yes — typed struct
pub fn create_user(req: CreateUserRequest) -> Result<User, CreateUserError> { ... }

// No — unstructured map
pub fn create_user(data: HashMap<String, String>) -> Result<User, CreateUserError> { ... }
```

- **Define clear types** for all data that crosses module boundaries. Exported
  structs and enums serve as the contract between modules.
- **Use `From`/`Into` traits** for type conversions at boundaries. This keeps
  conversion logic centralised and testable:

```rust
impl From<ApiResponse> for DomainModel {
    fn from(resp: ApiResponse) -> Self {
        Self {
            id: resp.id,
            name: resp.name,
        }
    }
}
```

- **At system boundaries**, deserialize into typed structs (via `serde`)
  immediately. Downstream code should never operate on raw `serde_json::Value`
  or unvalidated strings:

```rust
let config: Config = serde_json::from_str(&raw_json)?;
// use config (typed) from here on
```

- **Use newtypes** (`struct UserId(u64)`) to distinguish semantically different
  values of the same primitive type. The compiler will reject mixing a
  `UserId` with an `OrderId` even though both wrap `u64`.

### 5.6 SOLID in Rust

| Principle | Rust Application |
|---|---|
| **S** — Single Responsibility | One module per concern; one type per concept. |
| **O** — Open/Closed | Extend via new trait implementations and generics — not by editing existing code. |
| **L** — Liskov Substitution | Any type implementing a trait must honour its documented contract. |
| **I** — Interface Segregation | Keep traits small. Compose larger APIs from smaller traits (`Read + Write = ReadWrite`). |
| **D** — Dependency Inversion | Accept trait bounds (`impl Trait` or `dyn Trait`), not concrete types. Inject dependencies through constructors. |

---

## 6. Documentation & Comments

### 6.1 Doc Comments (*The Book* Ch. 14, Rust API Guidelines)

Rust uses `///` for item doc comments and `//!` for module/crate-level
documentation. Doc comments are **Markdown** and compiled into HTML by
`rustdoc`.

Every public item **must** have a doc comment:

```rust
/// A thread-safe counter backed by an atomic integer.
///
/// # Examples
///
/// ```
/// use mycrate::Counter;
///
/// let counter = Counter::new();
/// counter.increment();
/// assert_eq!(counter.get(), 1);
/// ```
pub struct Counter {
    value: AtomicU64,
}

impl Counter {
    /// Create a new counter initialised to zero.
    pub fn new() -> Self {
        Self {
            value: AtomicU64::new(0),
        }
    }

    /// Increment the counter by one.
    pub fn increment(&self) {
        self.value.fetch_add(1, Ordering::Relaxed);
    }

    /// Return the current count.
    pub fn get(&self) -> u64 {
        self.value.load(Ordering::Relaxed)
    }
}
```

### 6.2 Doc Comment Sections

Use these standard sections (Rust API Guidelines C-DOC):

| Section | Purpose | Required? |
|---|---|---|
| Summary line | One-line description starting with a verb | Always |
| Extended description | Detailed explanation, behaviour, invariants | When summary is insufficient |
| `# Examples` | Runnable code demonstrating usage | For all public items |
| `# Errors` | When and why the function returns `Err` | For all fallible functions |
| `# Panics` | Conditions under which the function panics | When applicable |
| `# Safety` | Invariants the caller must uphold | For all `unsafe fn` |

```rust
/// Parse a host:port string into its components.
///
/// # Examples
///
/// ```
/// let (host, port) = mycrate::parse_address("localhost:8080").unwrap();
/// assert_eq!(host, "localhost");
/// assert_eq!(port, 8080);
/// ```
///
/// # Errors
///
/// Returns `ParseError::InvalidFormat` if the input does not contain
/// exactly one colon, or `ParseError::InvalidPort` if the port is not
/// a valid `u16`.
pub fn parse_address(input: &str) -> Result<(&str, u16), ParseError> {
    ...
}
```

### 6.3 Module-Level Documentation

Use `//!` at the top of `lib.rs` or a module file:

```rust
//! # Authentication
//!
//! This module provides middleware for authenticating HTTP requests
//! using JWT tokens. It supports both symmetric (HS256) and asymmetric
//! (RS256) signing algorithms.

use jsonwebtoken::{decode, DecodingKey, Validation};
```

### 6.4 Code Comments (*Clean Code* Ch. 4)

- `//` comments explain **why**, not what.
- Never commit commented-out code.
- Use `// TODO:` and `// FIXME:` sparingly — track real issues in the issue
  tracker.
- `// SAFETY:` comments are **required** above every `unsafe` block,
  documenting why the operation is sound.

---

## 7. Error Handling

### 7.1 Principles (*The Book* Ch. 9)

> Use `Result<T, E>` for recoverable errors, `panic!` for unrecoverable bugs.

- **Never panic in library code** — return `Result` and let the caller decide.
- **Define domain error types** — don't return `Box<dyn Error>` from public
  APIs.
- **Use the `?` operator** for concise error propagation.

### 7.2 Error Type Hierarchy

Define a crate-level error enum:

```rust
use std::fmt;

/// Errors that can occur in the configuration subsystem.
#[derive(Debug)]
pub enum ConfigError {
    /// The configuration file could not be read.
    Io(std::io::Error),
    /// The configuration file contained invalid TOML.
    Parse(toml::de::Error),
    /// A required key was missing.
    MissingKey(&'static str),
}

impl fmt::Display for ConfigError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            Self::Io(e) => write!(f, "config I/O error: {e}"),
            Self::Parse(e) => write!(f, "config parse error: {e}"),
            Self::MissingKey(k) => write!(f, "missing config key: {k}"),
        }
    }
}

impl std::error::Error for ConfigError {
    fn source(&self) -> Option<&(dyn std::error::Error + 'static)> {
        match self {
            Self::Io(e) => Some(e),
            Self::Parse(e) => Some(e),
            Self::MissingKey(_) => None,
        }
    }
}

impl From<std::io::Error> for ConfigError {
    fn from(e: std::io::Error) -> Self {
        Self::Io(e)
    }
}
```

- Implement `Display`, `Error`, and `From` for each source error type.
- Consider using `thiserror` for derive-based boilerplate reduction.
- Use `anyhow` in applications (not libraries) for ad-hoc error context.

### 7.3 The `?` Operator

```rust
/// Load and parse the configuration file.
pub fn load_config(path: &Path) -> Result<Config, ConfigError> {
    let contents = fs::read_to_string(path)?;   // io::Error → ConfigError::Io
    let config: Config = toml::from_str(&contents)?;  // toml::de::Error → ConfigError::Parse
    if config.database_url.is_empty() {
        return Err(ConfigError::MissingKey("database_url"));
    }
    Ok(config)
}
```

### 7.4 `Option<T>` vs. `Result<T, E>`

| Use | When |
|---|---|
| `Option<T>` | Absence is **normal and expected** (lookup miss, optional field). |
| `Result<T, E>` | Absence is an **error** the caller must handle. |

Never use sentinel values (`-1`, empty strings) to represent absence.

### 7.5 `panic!`, `unwrap`, `expect`

- **Never `unwrap()`** in production code paths — use `expect()` with a
  descriptive message, or better yet, propagate the error.
- `expect("reason")` is acceptable in program initialisation or tests where
  failure is truly unrecoverable.
- `panic!` is for programming errors (violated invariants), not runtime
  failures.

---

## 8. Modules, Crates & Imports

### 8.1 Module Design

- A module provides **one concept** — `auth`, `storage`, `config`.
- Keep modules small. If a module file exceeds ~300 lines, consider splitting
  it into a directory with submodules.
- Use `pub(crate)` for items that are internal to the crate but shared across
  modules. Reserve `pub` for the external API.

### 8.2 Import Ordering

Group `use` declarations separated by blank lines:

1. **Standard library** (`std`, `core`, `alloc`)
2. **External crates** (`serde`, `tokio`, `anyhow`)
3. **Internal modules** (`crate::`, `super::`, `self::`)

```rust
use std::collections::HashMap;
use std::io::{self, Read};

use serde::{Deserialize, Serialize};
use tokio::sync::Mutex;

use crate::config::Config;
use crate::store::UserStore;
```

### 8.3 Import Style

- Use **nested imports** to reduce line count:

```rust
use std::io::{self, BufRead, Write};
```

- Import **types directly** when used frequently: `use std::path::PathBuf;`.
- Import **modules** when you want qualified access for clarity:
  `use std::fs;` then `fs::read_to_string(...)`.
- **Never** use glob imports (`use module::*`) except in test modules and
  preludes.

### 8.4 One Crate per Concern — No Redundant Dependencies

When multiple crates can accomplish the same task, **pick one and use it
consistently**:

```rust
// Bad — two crates for serialization in the same project
use serde_json;
use simd_json;

// Good — pick one
use serde_json;
```

Common violations:

| Overlapping crates | Pick one |
|---|---|
| `serde_json` / `simd-json` / `sonic-rs` | One JSON library per project |
| `reqwest` / `hyper` / `ureq` for HTTP clients | One HTTP client |
| `log` / `tracing` for diagnostics | `tracing` (superset of `log`) |
| `tokio` / `async-std` for async runtime | One async runtime per binary |
| `anyhow` / `eyre` for error context | One in applications |
| `chrono` / `time` for date/time | One time library per project |
| `clap` / `structopt` / `argh` for CLI args | One arg parser |
| `rand` / `fastrand` for random numbers | One RNG library unless benchmarking shows need |

### 8.5 Re-exports

Use re-exports in `lib.rs` to provide a flat, ergonomic public API while
keeping internal structure modular:

```rust
// lib.rs
pub mod config;
pub mod store;

// Re-export key types at the crate root
pub use config::Config;
pub use store::{Store, StoreError};
```

---

## 9. Ownership, Borrowing & Lifetimes

### 9.1 Core Rules (*The Book* Ch. 4)

- Every value has exactly **one owner**.
- When the owner goes out of scope, the value is **dropped**.
- You can have **either** one mutable reference **or** any number of immutable
  references — never both.

### 9.2 Borrowing Guidelines

| Situation | Guideline |
|---|---|
| Read-only access | Borrow with `&T` |
| Mutation needed | Borrow with `&mut T` |
| Transfer of ownership | Move or `clone()` |
| Long-lived shared access | `Arc<T>` (thread-safe) or `Rc<T>` (single-thread) |
| Interior mutability | `Cell<T>`, `RefCell<T>`, `Mutex<T>`, `RwLock<T>` |

### 9.3 Lifetime Annotations

- **Elide lifetimes** when the compiler's rules handle them (most of the time).
- Annotate explicitly only when the compiler requires it or when the
  relationship between input and output lifetimes is non-obvious:

```rust
/// Return the longer of two string slices.
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() >= y.len() { x } else { y }
}
```

- Name lifetimes descriptively when there are multiple:
  `'input`, `'output`, `'ctx` — not `'a`, `'b`, `'c`.

### 9.4 Clone and Copy

- **Derive `Copy`** for small, stack-only types (numbers, enum variants without
  data).
- **Derive `Clone`** for types that can be duplicated but involve heap
  allocation.
- Avoid cloning in hot paths — prefer borrowing.
- An unnecessary `.clone()` is a code smell worth investigating.

---

## 10. Concurrency

### 10.1 Principles (*The Book* Ch. 16)

> "Fearless concurrency."

The borrow checker prevents data races at compile time. Rust's concurrency
model is built on ownership:

- Types implementing `Send` can be transferred between threads.
- Types implementing `Sync` can be shared between threads via references.

### 10.2 Shared State

```rust
use std::sync::{Arc, Mutex};

/// A thread-safe hit counter.
#[derive(Clone)]
pub struct HitCounter {
    count: Arc<Mutex<u64>>,
}

impl HitCounter {
    /// Create a new counter starting at zero.
    pub fn new() -> Self {
        Self {
            count: Arc::new(Mutex::new(0)),
        }
    }

    /// Increment the counter by one.
    pub fn increment(&self) {
        let mut guard = self.count.lock().expect("mutex poisoned");
        *guard += 1;
    }

    /// Return the current count.
    pub fn get(&self) -> u64 {
        *self.count.lock().expect("mutex poisoned")
    }
}
```

- Use `Arc<Mutex<T>>` for shared mutable state across threads.
- Prefer `RwLock<T>` when reads vastly outnumber writes.
- Keep lock scopes as narrow as possible.

### 10.3 Message Passing

Use channels (`std::sync::mpsc` or crossbeam/tokio channels) when transferring
ownership of data between threads:

```rust
use std::sync::mpsc;
use std::thread;

/// Spawn a worker that processes jobs from a channel.
fn spawn_worker(rx: mpsc::Receiver<Job>) -> thread::JoinHandle<()> {
    thread::spawn(move || {
        for job in rx {
            process(job);
        }
    })
}
```

### 10.4 Async/Await

For I/O-bound concurrency, use `async`/`.await` with a runtime (e.g.
`tokio`):

```rust
/// Fetch all URLs concurrently and return their response bodies.
///
/// # Errors
///
/// Returns the first error encountered during fetching.
pub async fn fetch_all(urls: &[String]) -> Result<Vec<String>, reqwest::Error> {
    let client = reqwest::Client::new();
    let futures: Vec<_> = urls
        .iter()
        .map(|url| {
            let client = client.clone();
            let url = url.clone();
            async move { client.get(&url).send().await?.text().await }
        })
        .collect();

    futures::future::try_join_all(futures).await
}
```

- **Pick one async runtime** (`tokio`, `async-std`) and use it consistently.
- **Do not block** inside async functions — use `spawn_blocking` for CPU-heavy
  work.
- Minimise the `'static` bound surface by structuring lifetimes carefully.

### 10.5 Common Pitfalls

- **Deadlocks.** Acquire locks in a consistent order. Keep critical sections
  small.
- **Mutex poisoning.** Handle or document `expect("mutex poisoned")` calls.
- **Blocking in async.** Use `tokio::task::spawn_blocking` or
  `tokio::fs` for blocking I/O.

---

## 11. Design Patterns in Rust

The GoF patterns remain relevant, but Rust's ownership model, enums, traits,
and zero-cost abstractions change how they are expressed.

### 11.1 Creational Patterns

#### Constructor Functions

Rust has no built-in constructors. Use `new()` and associated functions:

```rust
/// An HTTP client with configurable timeout.
pub struct Client {
    base_url: String,
    timeout: Duration,
}

impl Client {
    /// Create a new client with the given base URL and a default 30s timeout.
    pub fn new(base_url: impl Into<String>) -> Self {
        Self {
            base_url: base_url.into(),
            timeout: Duration::from_secs(30),
        }
    }

    /// Create a client with a custom timeout.
    pub fn with_timeout(base_url: impl Into<String>, timeout: Duration) -> Self {
        Self {
            base_url: base_url.into(),
            timeout,
        }
    }
}
```

#### Builder

The Builder pattern is idiomatic Rust for complex construction:

```rust
/// Builds a `Server` with validated configuration.
pub struct ServerBuilder {
    addr: String,
    workers: usize,
    tls: bool,
}

impl ServerBuilder {
    /// Start building a server on the given address.
    pub fn new(addr: impl Into<String>) -> Self {
        Self {
            addr: addr.into(),
            workers: num_cpus::get(),
            tls: false,
        }
    }

    /// Set the number of worker threads.
    pub fn workers(mut self, n: usize) -> Self {
        self.workers = n;
        self
    }

    /// Enable TLS.
    pub fn tls(mut self, enabled: bool) -> Self {
        self.tls = enabled;
        self
    }

    /// Consume the builder and create the server.
    ///
    /// # Errors
    ///
    /// Returns an error if the address cannot be bound.
    pub fn build(self) -> Result<Server, ServerError> {
        ...
    }
}
```

- Builders consume `self` in each method (owned builder) to prevent reuse
  after `build()`.
- Use the `typed-builder` or `derive_builder` crates to reduce boilerplate when
  appropriate.

#### Singleton

Use `std::sync::OnceLock` (Rust 1.70+) for lazy, thread-safe initialisation:

```rust
use std::sync::OnceLock;

/// Return the global application config, initialised on first access.
fn global_config() -> &'static Config {
    static CONFIG: OnceLock<Config> = OnceLock::new();
    CONFIG.get_or_init(|| Config::from_env().expect("invalid config"))
}
```

### 11.2 Structural Patterns

#### Decorator (Wrapper)

Wrap a trait implementor to add behaviour:

```rust
/// A writer that counts the bytes written through it.
pub struct CountingWriter<W: Write> {
    inner: W,
    bytes_written: u64,
}

impl<W: Write> CountingWriter<W> {
    /// Wrap an existing writer.
    pub fn new(inner: W) -> Self {
        Self {
            inner,
            bytes_written: 0,
        }
    }

    /// Return the total number of bytes written so far.
    pub fn bytes_written(&self) -> u64 {
        self.bytes_written
    }
}

impl<W: Write> Write for CountingWriter<W> {
    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
        let n = self.inner.write(buf)?;
        self.bytes_written += n as u64;
        Ok(n)
    }

    fn flush(&mut self) -> io::Result<()> {
        self.inner.flush()
    }
}
```

#### Adapter

Implement a target trait by wrapping a foreign type:

```rust
/// Adapts a `LegacyPrinter` to the `Printer` trait.
pub struct PrinterAdapter {
    legacy: LegacyPrinter,
}

impl Printer for PrinterAdapter {
    fn print(&self, text: &str) {
        self.legacy.print_old(text);
    }
}
```

#### Newtype (Wrapper for Type Safety)

A zero-cost wrapper that prevents mixing up semantically different values:

```rust
/// Email address validated at construction time.
#[derive(Debug, Clone, PartialEq, Eq)]
pub struct Email(String);

impl Email {
    /// Parse and validate an email address.
    ///
    /// # Errors
    ///
    /// Returns `ValidationError` if the input is not a valid email.
    pub fn parse(input: &str) -> Result<Self, ValidationError> {
        if input.contains('@') {
            Ok(Self(input.to_owned()))
        } else {
            Err(ValidationError::invalid("email", input))
        }
    }
}
```

### 11.3 Behavioural Patterns

#### Strategy

Express strategies as trait objects or generics:

```rust
/// A sorting strategy.
pub trait Sorter {
    /// Sort the slice in place.
    fn sort(&self, data: &mut [i32]);
}

/// A collection that uses a pluggable sorting strategy.
pub struct DataSet<S: Sorter> {
    data: Vec<i32>,
    sorter: S,
}

impl<S: Sorter> DataSet<S> {
    /// Sort the internal data using the configured strategy.
    pub fn sort(&mut self) {
        self.sorter.sort(&mut self.data);
    }
}
```

Or simply use a closure for one-method strategies:

```rust
/// Sort data using a custom comparator.
pub fn sort_by(data: &mut [i32], cmp: impl Fn(&i32, &i32) -> std::cmp::Ordering) {
    data.sort_by(cmp);
}
```

#### Observer / Event System

```rust
use std::collections::HashMap;

type Callback = Box<dyn Fn(&str)>;

/// A simple event emitter.
pub struct EventEmitter {
    listeners: HashMap<String, Vec<Callback>>,
}

impl EventEmitter {
    /// Register a callback for the given event.
    pub fn on(&mut self, event: impl Into<String>, cb: impl Fn(&str) + 'static) {
        self.listeners
            .entry(event.into())
            .or_default()
            .push(Box::new(cb));
    }

    /// Emit an event, invoking all registered callbacks with the payload.
    pub fn emit(&self, event: &str, payload: &str) {
        if let Some(cbs) = self.listeners.get(event) {
            for cb in cbs {
                cb(payload);
            }
        }
    }
}
```

#### State Machine (Typestate)

Encode states as types so invalid transitions are compile errors:

```rust
/// A connection in the disconnected state.
pub struct Disconnected;
/// A connection in the connected state.
pub struct Connected;

/// A TCP connection parameterised by its state.
pub struct Connection<S> {
    addr: String,
    _state: std::marker::PhantomData<S>,
}

impl Connection<Disconnected> {
    /// Create a new, disconnected connection.
    pub fn new(addr: impl Into<String>) -> Self {
        Self {
            addr: addr.into(),
            _state: std::marker::PhantomData,
        }
    }

    /// Connect and transition to the Connected state.
    pub fn connect(self) -> Result<Connection<Connected>, io::Error> {
        // ... establish connection ...
        Ok(Connection {
            addr: self.addr,
            _state: std::marker::PhantomData,
        })
    }
}

impl Connection<Connected> {
    /// Send data over the connection.
    pub fn send(&self, data: &[u8]) -> io::Result<usize> {
        ...
    }
}
```

---

## 12. Testing

### 12.1 Principles

- Tests live in a `#[cfg(test)]` module at the bottom of each source file
  (unit tests) or in a top-level `tests/` directory (integration tests).
- **Test behaviour, not implementation.** Tests must survive refactors that
  preserve the public API.
- **Doc tests are real tests.** Every `# Examples` block in a doc comment is
  compiled and run.

### 12.2 Naming

Test functions are `snake_case` and describe the **scenario and expectation**:

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn withdraw_insufficient_funds_returns_error() {
        let mut account = Account::new(100);
        let result = account.withdraw(200);
        assert!(result.is_err());
    }

    #[test]
    fn parse_csv_empty_input_returns_empty_vec() {
        let result = parse_csv("");
        assert_eq!(result.unwrap(), Vec::<Record>::new());
    }
}
```

### 12.3 Assertion Style

- Use `assert_eq!` and `assert_ne!` with descriptive messages:

```rust
assert_eq!(account.balance(), 50, "balance should be reduced after withdrawal");
```

- Use `assert!(result.is_ok())` and `assert!(result.is_err())` for
  `Result` checks.
- For complex assertions, pattern-match or use `matches!`:

```rust
assert!(matches!(error, ConfigError::MissingKey("timeout")));
```

### 12.4 Test Organisation

| Test type | Location | Purpose |
|---|---|---|
| Unit tests | `#[cfg(test)] mod tests` in the source file | Test private and public functions in isolation |
| Integration tests | `tests/*.rs` | Test the public API as an external consumer |
| Doc tests | `///` comments | Verify examples compile and run |
| Benchmarks | `benches/*.rs` (with criterion) | Performance regression detection |

### 12.5 Test Utilities

- Extract common setup into helper functions — not macros, unless repetition
  is truly mechanical.
- Use `#[should_panic(expected = "...")]` for tests that verify panic messages.
- Use `proptest` or `quickcheck` for property-based testing when input spaces
  are large.

---

## 13. Database Access & ACID

When interacting with SQL databases, every query and transaction must respect
the **ACID** properties — **Atomicity**, **Consistency**, **Isolation**, and
**Durability**. Violations cause data corruption, phantom reads, and silent
data loss — bugs that are extraordinarily hard to diagnose after the fact.

### 13.1 ACID at a Glance

| Property | Guarantee | Violation Example |
|---|---|---|
| **Atomicity** | A transaction either completes entirely or has no effect. | A transfer debits one account but crashes before crediting the other. |
| **Consistency** | A transaction moves the database from one valid state to another; all constraints hold. | An insert succeeds despite violating a foreign key constraint. |
| **Isolation** | Concurrent transactions do not interfere with each other. | Two tasks read the same balance, both withdraw, and the final balance is wrong. |
| **Durability** | Once committed, data survives crashes and power loss. | A commit returns successfully but the data is lost after a restart. |

### 13.2 Always Use Explicit Transactions

Never rely on auto-commit for multi-statement operations. Wrap related writes
in a single transaction so they succeed or fail as a unit:

```rust
use sqlx::PgPool;

/// Transfer funds between two accounts atomically.
///
/// # Errors
///
/// Returns an error if either update fails or the transaction cannot commit.
pub async fn transfer(
    pool: &PgPool,
    from_id: i64,
    to_id: i64,
    amount: f64,
) -> Result<(), sqlx::Error> {
    let mut tx = pool.begin().await?;

    sqlx::query("UPDATE accounts SET balance = balance - $1 WHERE id = $2")
        .bind(amount)
        .bind(from_id)
        .execute(&mut *tx)
        .await?;

    sqlx::query("UPDATE accounts SET balance = balance + $1 WHERE id = $2")
        .bind(amount)
        .bind(to_id)
        .execute(&mut *tx)
        .await?;

    tx.commit().await
}
```

- `sqlx::Transaction` automatically rolls back on drop if not committed —
  leveraging Rust's RAII guarantee.
- Pass the transaction (`&mut *tx`) to all queries within the atomic unit.

### 13.3 Choose the Correct Isolation Level

Most databases default to READ COMMITTED. Set a stricter level when the
operation demands it:

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Use When |
|---|---|---|---|---|
| READ UNCOMMITTED | Possible | Possible | Possible | Almost never — analytics on stale data at most |
| READ COMMITTED | No | Possible | Possible | Default for most OLTP workloads |
| REPEATABLE READ | No | No | Possible | Reports or computations that must see a stable snapshot |
| SERIALIZABLE | No | No | No | Financial transactions, inventory, anything where correctness is paramount |

```rust
sqlx::query("SET TRANSACTION ISOLATION LEVEL SERIALIZABLE")
    .execute(&mut *tx)
    .await?;
```

Or with Diesel:

```rust
use diesel::connection::SimpleConnection;

conn.batch_execute("SET TRANSACTION ISOLATION LEVEL SERIALIZABLE")?;
```

### 13.4 Use Parameterised Queries — Never String Formatting

This protects both **consistency** (correct types) and **security** (SQL
injection). Rust's `sqlx` and `diesel` enforce this at compile time:

```rust
// Yes — parameterised (sqlx)
let user = sqlx::query_as::<_, User>("SELECT id, name FROM users WHERE id = $1")
    .bind(user_id)
    .fetch_one(pool)
    .await?;

// No — string formatting (SQL injection risk, bypasses type checking)
let query = format!("SELECT id, name FROM users WHERE id = {user_id}");
```

- **`sqlx`** checks queries against the database schema at compile time with
  `query!` and `query_as!`.
- **`diesel`** provides a fully type-safe DSL that makes SQL injection
  structurally impossible.

### 13.5 SQL Injection Protection

SQL injection remains one of the most dangerous vulnerabilities across all
languages. Even in Rust, where tools like `sqlx` and `diesel` provide strong
compile-time guarantees, developers must remain vigilant — especially when
writing raw SQL.

- **Always use parameterised queries** with placeholders (`$1`, `$2` for
  PostgreSQL, `?` for MySQL/SQLite). Never interpolate values into query
  strings.
- **Never use `format!()`** or string concatenation to build SQL statements:

```rust
// Bad — SQL injection via format!
let query = format!("SELECT * FROM users WHERE name = '{name}'");
sqlx::query(&query).fetch_all(pool).await?;

// Good — parameterised
sqlx::query("SELECT * FROM users WHERE name = $1")
    .bind(&name)
    .fetch_all(pool)
    .await?;
```

- **Use `sqlx::query!()` macro** for compile-time checked queries. The macro
  validates SQL syntax and parameter types against the actual database schema
  at build time, catching errors before they reach production.
- **Validate and constrain input** before it reaches the database layer.
  Restrict string lengths, check numeric ranges, and reject unexpected
  characters at the service boundary — not in the SQL query.

### 13.6 Handle Connection Lifecycle Properly

- **Use connection pools** (`sqlx::PgPool`, `diesel::r2d2`) — never open a
  connection per query.
- **Configure pool limits** (`max_connections`, `min_connections`,
  `idle_timeout`) based on workload and database limits.
- **Retry on transient errors** (connection reset, serialisation failure) with
  backoff, but never silently swallow the error.
- Leverage Rust's ownership model — a `Transaction` borrows the connection, so
  the compiler prevents using the connection outside the transaction while it
  is active.

### 13.7 ORM and Query Builder Transactions

When using Diesel, the same ACID principles apply:

```rust
use diesel::prelude::*;

/// Transfer funds between two accounts atomically.
///
/// # Errors
///
/// Returns an error if either update fails or the transaction cannot commit.
pub fn transfer(
    conn: &mut PgConnection,
    from_id: i64,
    to_id: i64,
    amount: f64,
) -> Result<(), diesel::result::Error> {
    conn.transaction::<_, diesel::result::Error, _>(|conn| {
        diesel::update(accounts::table.find(from_id))
            .set(accounts::balance.eq(accounts::balance - amount))
            .execute(conn)?;

        diesel::update(accounts::table.find(to_id))
            .set(accounts::balance.eq(accounts::balance + amount))
            .execute(conn)?;

        Ok(())
    })
}
```

- Diesel's `transaction` closure automatically rolls back if the closure
  returns `Err`.
- Use `SELECT ... FOR UPDATE` (via `.for_update()` in Diesel or raw SQL) to
  prevent concurrent modifications when reading values that will be updated
  within the same transaction.
- Keep transactions **short** — hold locks for the minimum time necessary.

---

## 14. Unsafe Code

### 14.1 Principles (*The Rustonomicon*)

> "Unsafe is an escape hatch, not a shortcut."

- `unsafe` does **not** disable the borrow checker. It only enables five
  additional operations: dereferencing raw pointers, calling unsafe functions,
  implementing unsafe traits, accessing/modifying mutable statics, and
  accessing fields of `union`s.
- **Minimise `unsafe` surface area.** Encapsulate it in a safe API.
- **Document every `unsafe` block** with a `// SAFETY:` comment explaining why
  the invariants are upheld.

### 14.2 Rules

**General prohibition:**
- Never use `unsafe` to work around a borrow checker error that could be
  solved with safe code.
- Every `unsafe fn` must have a `# Safety` section in its doc comment.
- Wrap unsafe operations in a safe abstraction and expose only the safe API.
- Test unsafe code with Miri (`cargo +nightly miri test`) to detect undefined
  behaviour.

**Strict exceptions for FFI and ABI interfaces:**
- `unsafe` is permitted for Foreign Function Interface (FFI) calls and
  system-level ABI compatibility when no safe alternative exists.
- Examples: C library bindings, system calls, memory-mapped I/O.
- FFI `unsafe` requires:
  - Exhaustive documentation of the C API contract being called
  - Clear explanation of why the invariants are upheld
  - Reference to the external C API documentation
  - Encapsulation in a safe wrapper whenever possible
  - Boundary validation before crossing into C code

```rust
/// A fixed-capacity stack allocated on the stack.
///
/// # Safety invariant
///
/// `len` is always ≤ `N`, and elements at indices `0..len` are initialised.
pub struct ArrayStack<T, const N: usize> {
    data: [MaybeUninit<T>; N],
    len: usize,
}

impl<T, const N: usize> ArrayStack<T, N> {
    /// Create a new, empty stack.
    pub const fn new() -> Self {
        // SAFETY: `MaybeUninit` is itself always valid in an uninitialised
        // state, so an array of `MaybeUninit<T>` requires no initialisation.
        Self {
            data: unsafe { MaybeUninit::uninit().assume_init() },
            len: 0,
        }
    }

    /// Push a value onto the stack.
    ///
    /// Returns `Err(value)` if the stack is full.
    pub fn push(&mut self, value: T) -> Result<(), T> {
        if self.len >= N {
            return Err(value);
        }
        self.data[self.len] = MaybeUninit::new(value);
        self.len += 1;
        Ok(())
    }

    /// Pop a value from the stack.
    pub fn pop(&mut self) -> Option<T> {
        if self.len == 0 {
            return None;
        }
        self.len -= 1;
        // SAFETY: elements at 0..(self.len + 1) were initialised by push();
        // after decrementing, index self.len is the last initialised slot.
        // assume_init_read moves the value out, logically uninitialising it.
        Some(unsafe { self.data[self.len].assume_init_read() })
    }
}
```

### 14.3 FFI and ABI Interface Pattern

When interfacing with C libraries or system APIs, encapsulate unsafe blocks
in a dedicated FFI module with a safe public API:

```rust
/// FFI bindings for a C library.
///
/// This module encapsulates the raw FFI declarations and provides
/// thin wrappers. Functions that require caller-upheld invariants
/// (valid handles, valid codes) are themselves `unsafe`; functions
/// that fully encapsulate the C contract internally are exposed as
/// safe.
pub mod ffi {
    use std::ffi::CStr;
    use libc::c_char;

    // C FFI declarations.
    // Contract: `c_function` is provided by [C library name]. It takes a valid
    // (non-null, not-yet-freed) handle and an error code, and returns a pointer
    // to a static null-terminated string owned by the library (never to be freed
    // by the caller). See: [link to official C API documentation].
    extern "C" {
        fn c_function(handle: *mut libc::c_void, code: libc::c_int) -> *const c_char;
    }

    /// Converts a C error code into a Rust string.
    ///
    /// # Safety
    ///
    /// - `handle` must be a valid pointer obtained from the C library and not
    ///   yet freed.
    /// - `code` must be a valid error code per the C API specification.
    ///
    /// Because the caller must uphold these invariants, this function is
    /// itself `unsafe`. A truly safe wrapper would accept a `Handle` newtype
    /// whose constructor enforces validity.
    pub unsafe fn error_to_string(handle: *mut libc::c_void, code: i32) -> String {
        // SAFETY: by the function's documented preconditions, `handle` is
        // valid and `code` is a recognised error code. `c_function` returns a
        // pointer to a static string owned by the C library that we only
        // read, never free. We null-check before constructing `CStr`, which
        // requires a non-null, null-terminated pointer.
        unsafe {
            let error_ptr = c_function(handle, code);
            if error_ptr.is_null() {
                "Unknown error".to_string()
            } else {
                CStr::from_ptr(error_ptr)
                    .to_string_lossy()
                    .into_owned()
            }
        }
    }
}
```

Key patterns:
- **Separate module:** `mod ffi` or `mod bindings` contains all FFI declarations
- **Validate at boundaries:** Check pointers are non-null, validate string 
  lengths, validate enums before passing to C
- **Document C contract:** Reference official C API docs (man pages, official 
  guides)
- **SAFETY comments:** Explain why the invariants are upheld for this specific call
- **Safe wrapper where possible:** prefer accepting safe newtypes that
  enforce invariants in their constructors so the resulting function is
  itself safe. When the caller must uphold invariants the function cannot
  verify, mark the function `pub unsafe fn` and document the preconditions
  in a `# Safety` section — never expose a safe-looking signature that
  silently relies on caller obligations

---

## 15. Performance & Idiomatic Rust

### 15.1 Iterators Over Loops

Prefer iterator chains — they are zero-cost abstractions:

```rust
// Yes
let sum: i32 = numbers.iter().filter(|n| **n > 0).sum();

// No
let mut sum = 0;
for n in &numbers {
    if *n > 0 {
        sum += n;
    }
}
```

But keep chains readable — if a chain exceeds three or four combinators,
extract helper functions or use intermediate `let` bindings.

### 15.2 `match` and `if let`

Use `match` for exhaustive handling. Use `if let` when you only care about one
variant:

```rust
// Exhaustive — compiler catches missing arms
match state {
    State::Ready => process(),
    State::Loading { progress } => show_progress(progress),
    State::Error(e) => log_error(e),
}

// Only one variant matters
if let Some(user) = find_user(id) {
    greet(&user);
}
```

### 15.3 Avoid Unnecessary Allocation

- Return `&str` instead of `String` when possible.
- Use `Cow<'_, str>` when a function sometimes borrows and sometimes owns.
- Pre-allocate collections: `Vec::with_capacity(n)`,
  `HashMap::with_capacity(n)`.
- Use `&[T]` parameters instead of `&Vec<T>`.

### 15.4 Smart Pointers

| Type | Use case |
|---|---|
| `Box<T>` | Single-owner heap allocation, recursive types, trait objects |
| `Rc<T>` | Shared ownership, single-threaded |
| `Arc<T>` | Shared ownership, multi-threaded |
| `Cow<'a, T>` | Clone-on-write; borrow when possible, clone when needed |

### 15.5 Standard Trait Implementations

Derive or implement these traits for all public types when semantically
appropriate:

- `Debug` — always
- `Clone` — when the type can be duplicated
- `Default` — when a meaningful default exists
- `Display` — for user-facing output
- `PartialEq` / `Eq` — for comparisons
- `Hash` — for use as map keys
- `From` / `Into` — for type conversions
- `Send` / `Sync` — verified automatically; document if intentionally not
  implemented

### 15.6 Guard Clauses

Return early rather than nesting deeply:

```rust
/// Process the order and return a receipt.
///
/// # Errors
///
/// Returns `OrderError::Cancelled` if the order has been cancelled.
pub fn process_order(order: &Order) -> Result<Receipt, OrderError> {
    if order.is_cancelled() {
        return Err(OrderError::Cancelled(order.id()));
    }
    if order.items().is_empty() {
        return Ok(Receipt::empty());
    }
    build_receipt(order)
}
```

---

## 16. Defensive Programming & Input Validation

Rust's type system and borrow checker provide stronger compile-time guarantees
than most languages, but they cannot validate the *content* of external input.
Strings can still be too long, numbers can still be out of range, and file
paths can still traverse directories. Validate at the boundary where data
enters the system.

### 16.1 Validate at System Boundaries

- **Validate all external input** at system boundaries — HTTP request handlers,
  CLI argument parsing, config file loading, and deserialization entry points.
  The type system handles the rest once data is inside the application.
- **Use newtypes with validation in constructors** to ensure invalid values
  cannot exist:

```rust
/// A validated TCP/UDP port number.
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub struct Port(u16);

impl Port {
    /// Create a new port, rejecting port 0.
    ///
    /// # Errors
    ///
    /// Returns `ValidationError` if the port is 0.
    pub fn new(n: u16) -> Result<Self, ValidationError> {
        if n == 0 {
            return Err(ValidationError::new("port must be non-zero"));
        }
        Ok(Self(n))
    }
}
```

- **Use `TryFrom`/`FromStr` traits** for validated conversions from raw types:

```rust
impl TryFrom<u16> for Port {
    type Error = ValidationError;

    fn try_from(value: u16) -> Result<Self, Self::Error> {
        Self::new(value)
    }
}
```

### 16.2 Make Illegal States Unrepresentable

- **Use enums** to constrain value sets at the type level. When only three
  statuses are valid, define an enum with three variants — not a `String`:

```rust
#[derive(Debug, Clone, PartialEq, Eq)]
pub enum OrderStatus {
    Pending,
    Shipped,
    Delivered,
}
```

- **Validate string lengths and patterns** at parsing boundaries. Once data
  passes validation and enters a newtype, downstream code need not re-check.

### 16.3 Leverage the Type System

- The **borrow checker and type system** provide most defensive guarantees
  automatically — null safety via `Option`, error propagation via `Result`,
  and data race prevention via `Send`/`Sync`. Lean into these rather than
  adding manual runtime checks for issues the compiler already prevents.

### 16.4 Input Sanitisation

- **Sanitize input used in file paths** with `std::path::Path::canonicalize`
  to resolve symlinks and relative components. Verify the resolved path is
  within the expected directory.
- **Sanitize input used in URLs** and command execution. Use the `url` crate
  for URL parsing and validation.
- **Never use `unsafe`** to bypass validation. `unsafe` is for low-level
  operations that the compiler cannot verify — not for skipping checks.

### 16.5 Error Discipline

- **Use `#[must_use]`** on `Result` types to prevent silently ignoring errors.
  The compiler will warn if a `#[must_use]` value is discarded:

```rust
#[must_use]
pub fn validate_config(config: &Config) -> Result<(), ConfigError> {
    ...
}
```

---

## 17. Project Structure & Cargo

### 17.1 Recommended Layout

```
my_project/
    Cargo.toml
    Cargo.lock               # committed for binaries, not for libraries
    rustfmt.toml
    clippy.toml
    src/
        main.rs              # binary entry point (or lib.rs for libraries)
        lib.rs               # library root (if both binary and library)
        config.rs
        models/
            mod.rs
            user.rs
            order.rs
        services/
            mod.rs
            auth.rs
    tests/
        integration_test.rs  # integration tests
    benches/
        benchmark.rs         # benchmarks (criterion)
    examples/
        basic_usage.rs       # runnable examples (cargo run --example)
```

### 17.2 Cargo.toml

```toml
[package]
name = "my-app"
version = "0.1.0"
edition = "2024"
rust-version = "1.85"  # Required for edition = "2024"

[dependencies]
serde = { version = "1", features = ["derive"] }
tokio = { version = "1", features = ["full"] }

[dev-dependencies]
criterion = { version = "0.5", features = ["html_reports"] }
proptest = "1"

[lints.rust]
unsafe_code = "forbid"

[lints.clippy]
all = { level = "warn", priority = -1 }
pedantic = { level = "warn", priority = -1 }
```

### 17.3 Dependency Management

- Specify the **minimum required version** in the manifest and let
  `Cargo.lock` pin the exact resolved version: `serde = "1.0"` is the caret
  requirement `>=1.0, <2.0`. Pin patch versions only when a specific bug fix is
  required.
- Separate `[dependencies]` from `[dev-dependencies]`.
- Audit dependencies with `cargo audit` in CI.
- Minimise feature flags — only enable what you use.
- Commit `Cargo.lock` for binaries; omit it for libraries (let downstream
  resolve).

### 17.4 Feature Flags

Use feature flags for optional capabilities:

```toml
[dependencies]
serde = { version = "1", features = ["derive"], optional = true }

[features]
default = []
serde-support = ["dep:serde"]
```

- Feature names use `kebab-case`.
- Features should be **additive** — enabling a feature should never break
  existing functionality.

---

## 18. Tooling

### 18.1 Recommended Tool Chain

| Purpose | Tool | Notes |
|---|---|---|
| Formatter | `rustfmt` | Non-negotiable — `cargo fmt` |
| Linter | `clippy` | Run with `cargo clippy -- -D warnings` |
| Test runner | `cargo test` | Includes unit, integration, and doc tests |
| Benchmarking | `criterion` | Statistical benchmarking in `benches/` |
| Security audit | `cargo audit` | Check for known vulnerabilities |
| Unsafe detection | `cargo +nightly miri test` | Detect undefined behaviour |
| Coverage | `cargo tarpaulin` or `llvm-cov` | Line and branch coverage |
| Documentation | `cargo doc --open` | Verify doc comments render correctly |
| Dependency check | `cargo deny` | License compliance, duplicate deps, advisories |

### 18.2 CI Checks

At minimum, CI should run:

```bash
cargo fmt -- --check         # formatting
cargo clippy -- -D warnings  # linting
cargo test                   # all tests (unit + integration + doc)
cargo audit                  # vulnerability scanning
cargo doc --no-deps          # documentation builds without errors
```

### 18.3 Clippy Configuration

Configure Clippy in `Cargo.toml` or `clippy.toml`:

```toml
# Cargo.toml
[lints.clippy]
all = { level = "warn", priority = -1 }
pedantic = { level = "warn", priority = -1 }
nursery = { level = "warn", priority = -1 }
unwrap_used = "deny"
expect_used = "warn"
```

---

## 19. Build Tools

### 19.1 Cargo (Native & Complete)

Cargo handles compilation, testing, documentation, and packaging:

```bash
cargo new myapp  # Create new binary project
cargo new --lib mylib  # Create library
cargo build  # Debug build
cargo build --release  # Optimized release build
cargo run  # Build and run
cargo test  # Run tests
cargo doc --open  # Generate and open documentation
cargo publish  # Publish to crates.io
```

### 19.2 Cargo.toml Configuration

```toml
[package]
name = "myapp"
version = "0.1.0"
edition = "2024"  # Rust edition: 2015, 2018, 2021, 2024 (stable since 1.85)

[dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1.0", features = ["full"] }

[dev-dependencies]
tokio-test = "0.4"

[[bin]]
name = "myapp"
path = "src/main.rs"

[profile.release]
opt-level = 3  # Maximum optimization
lto = true  # Link-time optimization
codegen-units = 1  # Single codegen unit for better optimization
```

### 19.3 Feature Flags

Conditional compilation with features:

```toml
[features]
default = ["std"]
std = []  # Standard library support
async-runtime = ["dep:tokio"]  # Optional async runtime (tokio must be declared optional)
```

Build with features:
```bash
cargo build --features async-runtime
cargo build --no-default-features  # Disable defaults
cargo build --all-features  # Enable all features
```

### 19.4 Workspaces

Organize multiple crates in one project:

```toml
# Workspace root Cargo.toml
[workspace]
members = ["crate-a", "crate-b", "utils"]

# Build all crates
cargo build --workspace
cargo test --workspace
```

### 19.5 Custom Build Scripts

`build.rs` for compile-time code generation. Build-time dependencies must be
declared under `[build-dependencies]`. Use Cargo's `cargo::` directives
(prefer the newer `cargo::` form on Rust 1.77+; `cargo:` is the legacy form):

```rust
// build.rs
use std::time::{SystemTime, UNIX_EPOCH};

fn main() {
    let secs = SystemTime::now()
        .duration_since(UNIX_EPOCH)
        .expect("clock before epoch")
        .as_secs();
    println!("cargo::rustc-env=BUILD_TIME_UNIX={secs}");
    println!("cargo::rerun-if-changed=build.rs");
}
```

### 19.6 Cross-platform Builds

Build for different targets:
```bash
rustup target add x86_64-pc-windows-gnu
cargo build --target x86_64-pc-windows-gnu

# Or use cross tool for easier cross-compilation
cross build --target aarch64-unknown-linux-gnu
```

---

## 20. SBOM Creation

### 20.1 What is an SBOM?

A Software Bill of Materials lists all dependencies and transitive crates. Essential for supply chain security, vulnerability tracking, and license compliance.

### 20.2 cargo-sbom and CycloneDX

**Using `cargo-cyclonedx`**:
```bash
cargo install cargo-cyclonedx
cargo cyclonedx --output sbom.json
cargo cyclonedx --output sbom.xml --format xml
```

### 20.3 Cargo.lock and Dependency Management

- `Cargo.lock`: Records exact dependency versions for reproducible builds
- Include in VCS for binary crates; optional for libraries
- `cargo tree`: View dependency tree
  ```bash
  cargo tree --duplicates  # Show duplicate transitive deps
  ```

### 20.4 Vulnerability Scanning

**Using `cargo-audit`** (checks RustSec advisory database):
```bash
cargo install cargo-audit
cargo audit  # Scans for known vulnerabilities
cargo audit --json > audit-report.json
```

### 20.5 License Compliance

**Using `cargo-license`**:
```bash
cargo install cargo-license
cargo license --json > licenses.json
cargo license --threshold 1  # Warn if many unknown licenses
```

### 20.6 Integration into CI/CD

- Run `cargo audit` as a pre-commit or CI gate
- Generate SBOM with `cargo-cyclonedx` on release
- Use `cargo-deny` to enforce license policies and block unsafe patterns:
  ```bash
  cargo install cargo-deny
  cargo deny check licenses  # Verify license compliance
  cargo deny check advisories  # Check for known vulnerabilities
  ```
- Store SBOM and audit results as release artifacts
- Monitor RustSec feed for updates

---

## 21. References

### Official Documentation

| Resource | URL |
|---|---|
| *The Rust Programming Language* ("The Book") | https://doc.rust-lang.org/book/ |
| *The Rustonomicon* | https://doc.rust-lang.org/nomicon/ |
| *Rust by Example* | https://doc.rust-lang.org/rust-by-example/ |
| Rust API Guidelines | https://rust-lang.github.io/api-guidelines/ |
| Rust Reference | https://doc.rust-lang.org/reference/ |
| Standard Library Docs | https://doc.rust-lang.org/std/ |
| Rust RFC Repository | https://rust-lang.github.io/rfcs/ |
| Clippy Lints | https://rust-lang.github.io/rust-clippy/ |
| Rustfmt Style Guide | https://rust-lang.github.io/rustfmt/ |

### Books

| Book | Authors | Key Takeaways for This Guide |
|---|---|---|
| *The Rust Programming Language* | Klabnik & Nichols | Ownership, borrowing, lifetimes, error handling, traits — the foundation. |
| *The Rustonomicon* | Rust Team | Unsafe code rules, raw pointers, FFI — when and how to escape safe Rust. |
| *Rust for Rustaceans* | Jon Gjengset (2021) | Advanced patterns: type system tricks, unsafe abstractions, macros, async. |
| *Programming Rust* | Blandy, Orendorff & Tindall (2021) | Comprehensive coverage with emphasis on ownership model and systems programming. |
| *Design Patterns: Elements of Reusable Object-Oriented Software* | Gamma, Helm, Johnson, Vlissides (1994) | Favour composition over inheritance; program to interfaces — traits in Rust are the natural fit. |
| *Clean Code: A Handbook of Agile Software Craftsmanship* | Robert C. Martin (2008) | Small functions, meaningful names, SRP — applies regardless of paradigm. |
| *Zero To Production In Rust* | Luca Palmieri (2022) | Real-world Rust: project structure, error handling, testing, observability. |
| *Rust Design Patterns* | Rust Community | https://rust-unofficial.github.io/patterns/ — idiomatic patterns and anti-patterns. |
