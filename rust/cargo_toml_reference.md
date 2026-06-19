# Rust Cargo.toml Configuration Reference

Guide for configuring `Cargo.toml` for binary and library projects with optimal settings for development and release builds.

---

## Binary Projects

Use this template for executable/CLI applications.

```toml
[package]
name = "my-app"
version = "0.1.0"
edition = "2024"  # stable since Rust 1.85 (Feb 2025); use "2021" for older toolchains
authors = ["Your Name <you@example.com>"]
description = "Brief description"
license = "MIT"
repository = "https://github.com/username/my-app"
keywords = ["key1", "key2"]
categories = ["command-line-utilities"]

[dependencies]
# Add your dependencies here
# tokio = { version = "1.0", features = ["full"] }
# serde = { version = "1.0", features = ["derive"] }

[dev-dependencies]
# Test dependencies only
# tokio-test = "0.4"

[profile.release]
# Production optimizations
strip = true              # Strip symbols
lto = true               # Link-time optimization
opt-level = 3            # Maximum optimization
codegen-units = 1        # Single codegen unit (slower build, better optimization)
panic = "abort"          # Abort on panic (smaller binary)

[profile.dev]
# Development build settings
codegen-units = 32       # Fast compilation

[[bin]]
name = "main"
path = "src/main.rs"

# Optional: additional binaries
# [[bin]]
# name = "subcommand"
# path = "src/bin/subcommand.rs"
```

### Key Binary Settings

| Setting | Purpose | Development | Release |
|---------|---------|-------------|---------|
| `strip` | Remove debug symbols | false | true |
| `lto` | Link-time optimization | false | true |
| `opt-level` | Optimization level (0-3, s, z) | 0 (fastest build) | 3 (fastest runtime) |
| `codegen-units` | Parallel compilation units | 32 (faster) | 1 (better optimization) |
| `panic` | Panic behavior | unwind | abort |

---

## Library Projects

Use this template for reusable libraries published to crates.io.

```toml
[package]
name = "my-lib"
version = "0.1.0"
edition = "2024"  # stable since Rust 1.85 (Feb 2025); use "2021" for older toolchains
authors = ["Your Name <you@example.com>"]
description = "Brief description of the library"
documentation = "https://docs.rs/my-lib"
repository = "https://github.com/username/my-lib"
homepage = "https://github.com/username/my-lib"
keywords = ["key1", "key2"]
categories = ["data-structures", "algorithms"]
license = "MIT"
readme = "README.md"

[lib]
name = "my_lib"
path = "src/lib.rs"

[dependencies]
# Public dependencies (appear in user's transitive deps)
# tokio = { version = "1.0", features = ["full"] }
# serde = { version = "1.0", features = ["derive"] }

[dev-dependencies]
# Only for tests and examples
# tokio-test = "0.4"

[features]
default = []
# Feature flags for conditional compilation
# std = []          # Standard library support (optional)
# async-runtime = ["dep:tokio"]  # Optional async runtime (use dep: prefix)
# full = ["std", "async-runtime"]

[profile.release]
# Production optimizations
strip = true              # Strip symbols
lto = true               # Link-time optimization
opt-level = 3            # Maximum optimization
codegen-units = 1        # Single codegen unit
panic = "abort"          # Abort on panic

[profile.dev]
# Development build settings
codegen-units = 32       # Fast compilation

# Optional: additional binary targets (examples, CLI)
# [[bin]]
# name = "example"
# path = "examples/example.rs"
```

### Key Library Settings

| Setting | Purpose | Notes |
|---------|---------|-------|
| `[lib]` | Library target definition | Defaults to `src/lib.rs` |
| `documentation` | docs.rs URL | Users can browse docs online |
| `readme` | Displayed on crates.io | Usually `README.md` |
| `homepage` | Project homepage link | Optional but recommended |
| `[features]` | Feature flags | Users can enable/disable features |

---

## Feature Flags (Libraries)

Enable optional functionality without pulling in unnecessary dependencies:

```toml
[features]
default = ["std"]

std = []                    # Standard library support
async = ["dep:tokio"]      # Async runtime (tokio declared as optional dep)
serde = ["dep:serde"]      # Serialization
full = ["std", "async", "serde"]

# docs.rs build configuration
[package.metadata.docs.rs]
features = ["full"]  # Enable the "full" feature when docs.rs builds documentation
```

Usage in code:

```rust
#[cfg(feature = "serde")]
use serde::{Serialize, Deserialize};

#[cfg(feature = "async")]
pub async fn async_function() { }
```

---

## Common Dependency Configurations

### Pinning Versions

```toml
[dependencies]
# Default is caret-equivalent: allows minor/patch updates within the same major
tokio = "1.35.0"   # allows >=1.35.0, <2.0.0  (same as "^1.35.0")
tokio = "1.35"     # allows >=1.35.0, <2.0.0
tokio = "1"        # allows >=1.0.0, <2.0.0

# Exact version (requires explicit `=` prefix)
tokio = "=1.35.0"  # only 1.35.0

# Tilde — patch-level updates only
tokio = "~1.35.0"  # allows >=1.35.0, <1.36.0

# Specific version range
tokio = ">=1.34, <=1.35"
```

Note: `Cargo.lock` records the exact resolved version for reproducible builds;
the manifest specifies the *allowed range*.

### Optional Dependencies

```toml
[dependencies]
serde = { version = "1.0", optional = true }
tokio = { version = "1.0", optional = true }

[features]
serialization = ["dep:serde"]
async-support = ["dep:tokio"]
```

### Feature-Gated Dependencies

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1.0", features = ["rt-multi-thread", "macros"] }

# dep: prefix for optional dependencies
[dependencies]
serde = { version = "1.0", optional = true }

[features]
with-serde = ["dep:serde"]
```

---

## Profile Configurations

### Development vs Release

| Profile | Use Case | Focus | Optimization |
|---------|----------|-------|--------------|
| `dev` | `cargo build`, testing | Fast build time | Minimal |
| `release` | `cargo build --release`, production | Fast execution | Maximum |
| `test` | `cargo test` | Fast build + accurate results | Balanced |
| `bench` | `cargo bench` | Fast execution | Maximum |

### Common Profile Customizations

```toml
# Fast development builds
[profile.dev]
opt-level = 0
codegen-units = 32

# Fast release builds (trade optimization for speed)
[profile.release]
opt-level = 2
lto = "thin"
codegen-units = 16

# Minimal binary size
[profile.release]
opt-level = "z"
lto = true
codegen-units = 1
strip = true
panic = "abort"
```

---

## Edition Differences

| Edition | Year | Key Features | Recommended |
|---------|------|--------------|-------------|
| 2015 | 2015 | Original Rust | No, legacy only |
| 2018 | 2018 | Module path changes, `async`/`await` keywords reserved | Only if maintaining old code |
| 2021 | 2021 | Disjoint closure captures, `IntoIterator` for arrays, panic macro consistency | Acceptable for existing projects |
| 2024 | 2024 | Stabilised in Rust 1.85 (Feb 2025). RPIT lifetime capture, unsafe `extern` blocks, `if let` temporary scopes, reserves the `gen` keyword (gen blocks themselves remain nightly-only behind `#![feature(gen_blocks)]`) | Yes, recommended for new projects |

Use `edition = "2024"` for new projects.

---

## Publishing to crates.io

Required metadata:

```toml
[package]
name = "my-lib"                                # crates.io identifier
version = "0.1.0"                              # Semantic versioning
authors = ["Your Name <email@example.com>"]   # Author info
description = "What the crate does"           # Short description
license = "MIT"                                # License identifier
repository = "https://github.com/user/repo"   # Source code link

# Recommended
documentation = "https://docs.rs/my-lib"
homepage = "https://example.com"
readme = "README.md"
keywords = ["web", "async"]                   # Search tags (max 5)
categories = ["web-programming"]              # Categories
```

Publish:

```bash
cargo publish --dry-run  # Test publish
cargo publish            # Publish to crates.io
cargo publish --allow-dirty  # Allow uncommitted changes (not recommended)
```

---

## Workspaces

For multi-crate projects:

```toml
# Root Cargo.toml
[workspace]
members = ["crate-a", "crate-b", "shared"]
resolver = "2"

[workspace.package]
# Shared version and metadata
version = "0.1.0"
authors = ["Your Name"]
edition = "2024"

# Per-crate Cargo.toml references shared values
[package]
name = "crate-a"
version.workspace = true
authors.workspace = true
edition.workspace = true
```

Build entire workspace:

```bash
cargo build --workspace
cargo test --workspace
# Publish individual crates (Cargo does not publish a whole workspace at once
# because crates.io accepts one crate per publish; publish dependencies first):
cargo publish -p shared
cargo publish -p crate-a
cargo publish -p crate-b
```

---

## Best Practices

### Binary Projects
- ✅ Set `panic = "abort"` for smaller binaries
- ✅ Enable `lto = true` and `strip = true` for release builds
- ✅ Use `opt-level = 3` for maximum performance
- ✅ Keep `codegen-units = 32` in dev for fast iteration
- ❌ Don't publish binaries as libraries to crates.io

### Library Projects
- ✅ Keep dependencies minimal; users will pull them transitively
- ✅ Use feature flags for optional functionality
- ✅ Provide comprehensive documentation and examples
- ✅ Version conservatively using semver
- ❌ Don't include dev-only dependencies in `[dependencies]`

### All Projects
- ✅ Lock `Cargo.lock` in VCS for binaries; optional for libraries
- ✅ Use `resolver = "2"` in workspaces
- ✅ Pin major versions explicitly; allow minor/patch to float
- ✅ Test with `cargo test --all-features`
- ❌ Don't hard-code absolute paths; use relative paths or `env!("CARGO_MANIFEST_DIR")`

---

## Quick Reference

**Binary Project:**
```toml
[package]
edition = "2024"

[dependencies]
# Your deps here

[profile.release]
strip = true
lto = true
opt-level = 3
codegen-units = 1
panic = "abort"

[profile.dev]
codegen-units = 32

[[bin]]
name = "main"
path = "src/main.rs"
```

**Library Project:**
```toml
[package]
edition = "2024"

[lib]
path = "src/lib.rs"

[dependencies]
# Your deps here

[features]
default = []

[profile.release]
strip = true
lto = true
opt-level = 3
codegen-units = 1
panic = "abort"

[profile.dev]
codegen-units = 32
```
