# Rust Cargo.toml — AI Instructions

Apply these directives when generating or modifying `Cargo.toml` for Rust projects.

## Quick Decision: Binary or Library?

- **Binary project** (`src/main.rs`): Application, CLI tool, executable. Include `[[bin]]` section. Optimize for performance and size in release.
- **Library project** (`src/lib.rs`): Reusable crate published to crates.io. Include `[lib]` section. Use feature flags for optional functionality.

## Essential Boilerplate (All Projects)

```toml
[package]
name = "my-project"
version = "0.1.0"
edition = "2024"  # Stable since Rust 1.85 (Feb 2025); use "2021" for older toolchains
authors = ["Author Name <email@example.com>"]
description = "Brief description"
license = "MIT"
repository = "https://github.com/user/repo"

[dependencies]
# Your dependencies here

[profile.release]
strip = true              # Remove debug symbols
lto = true               # Link-time optimization
opt-level = 3            # Maximum optimization
codegen-units = 1        # Single codegen unit for better optimization
panic = "abort"          # Abort on panic (smaller binary)

[profile.dev]
codegen-units = 32       # Fast compilation during development
```

## Binary Project Specifics

```toml
[[bin]]
name = "main"
path = "src/main.rs"

# Optional: additional binaries
# [[bin]]
# name = "subcommand"
# path = "src/bin/subcommand.rs"
```

## Library Project Specifics

```toml
[lib]
name = "my_lib"
path = "src/lib.rs"

[features]
default = []
# feature_name = []
# feature_with_deps = ["dependency"]
# full = ["feature_name", "feature_with_deps"]  # Define before referencing in docs.rs metadata

# Metadata for docs.rs
[package.metadata.docs.rs]
all-features = true  # Or list explicitly: features = ["full"] — must match defined features

# Additional metadata for crates.io
documentation = "https://docs.rs/my-lib"
homepage = "https://github.com/user/repo"
readme = "README.md"
keywords = ["key1", "key2"]  # Max 5, for search
categories = ["category-name"]
```

## Dependency Version Pinning

- **Caret (default)**: `tokio = "1.35.0"` allows `>=1.35.0, <2.0.0` (same as `"^1.35.0"`)
- **Shorter form**: `tokio = "1.35"` — also caret semantics, allows `>=1.35.0, <2.0.0`
- **Exact match**: `tokio = "=1.35.0"` — only `1.35.0` (use `=` prefix; rarely necessary)
- **Tilde**: `tokio = "~1.35.0"` allows `>=1.35.0, <1.36.0` (patch updates only)
- **Range**: `tokio = ">=1.34, <=1.35"` — explicit bounds
- **Pin major versions explicitly**; allow minor/patch to float
- **Note**: `Cargo.lock` pins the exact resolved version for reproducible builds; the manifest specifies the *allowed range*.

## Feature Flags (Libraries Only)

```toml
[dependencies]
serde = { version = "1.0", optional = true }
tokio = { version = "1.0", optional = true }

[features]
default = []
serialization = ["dep:serde"]
async-support = ["dep:tokio"]
full = ["serialization", "async-support"]
```

Use `dep:` prefix to bind optional dependencies to features. In code:
```rust
#[cfg(feature = "serialization")]
use serde::{Serialize, Deserialize};
```

## Dependency Scope

- `[dependencies]`: public dependencies (users will pull transitively)
- `[dev-dependencies]`: test-only (tokio-test, proptest, etc.)
- `[build-dependencies]`: build-time only (build.rs scripts)

## Workspace (Multi-Crate Projects)

```toml
[workspace]
members = ["crate-a", "crate-b", "shared"]
resolver = "2"

[workspace.package]
version = "0.1.0"
authors = ["Name"]
edition = "2024"

# In each member's Cargo.toml:
[package]
name = "crate-a"
version.workspace = true
authors.workspace = true
edition.workspace = true
```

## Profile Customization

### Fast Development Builds
```toml
[profile.dev]
opt-level = 0
codegen-units = 32
```

### Fast Release Builds (Trade Some Optimization)
```toml
[profile.release]
opt-level = 2
lto = "thin"
codegen-units = 16
```

### Minimal Binary Size
```toml
[profile.release]
opt-level = "z"
lto = true
codegen-units = 1
strip = true
panic = "abort"
```

## Publishing to crates.io

Required metadata:
```toml
[package]
name = "unique-crate-name"
version = "0.1.0"
authors = ["Your Name <email@example.com>"]
description = "What your crate does"
license = "MIT"  # or other SPDX identifier
repository = "https://github.com/user/repo"
documentation = "https://docs.rs/crate-name"
readme = "README.md"
keywords = ["web", "async"]
categories = ["web-programming"]
```

Commands:
```bash
cargo publish --dry-run  # Test publish
cargo publish            # Publish to crates.io
```

## Common Pitfalls to Avoid

- ❌ Don't mix feature-gated and unconditional dependencies — users will pull all of them.
- ❌ Don't publish binaries as libraries to crates.io.
- ❌ Don't forget to pin `Cargo.lock` in VCS for binary projects.
- ❌ Don't use wildcard versions (`*`) or `latest` — always pin at least major version.
- ❌ Don't include dev-only dependencies in `[dependencies]`.
- ❌ Don't hard-code absolute paths — use `env!("CARGO_MANIFEST_DIR")`.

## Linting & Policy Enforcement

For FFI/ABI projects (C library bindings, system integration):

```toml
# In Cargo.toml — relax unsafe restriction for legitimate FFI
[lints.rust]
# Allow unsafe blocks in strict limits for FFI, but still enforce clippy warnings
unsafe_code = "warn"  # Instead of "forbid" for FFI projects
```

`cargo-deny` is configured in a separate `deny.toml` file at the project root,
not under `[package.metadata]`. Example `deny.toml`:

```toml
[advisories]
yanked = "deny"
unmaintained = "warn"

[licenses]
allow = ["MIT", "Apache-2.0", "BSD-3-Clause"]
```

For standard projects (binaries, libraries without system-level integration):

```toml
[lints.rust]
unsafe_code = "forbid"  # No unsafe allowed
```

Use `cargo-deny` to audit dependencies for unsafe patterns and advisory vulnerabilities.

## For Full Details

Refer to `cargo_toml_reference.md` for:
- Detailed explanations of all settings
- Profile optimisation scenarios
- Feature flag patterns
- Workspace strategies
- Best practices by project type
