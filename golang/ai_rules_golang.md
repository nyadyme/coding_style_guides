# Go — AI Coding Rules

Apply these rules when generating or reviewing Go code.

## Formatting

- `gofmt`/`goimports` is non-negotiable. Tabs for indentation.
- Soft line limit: 80-100 characters.
- One blank line between top-level definitions. No more than one.
- No blank line at start/end of function body.
- One blank line between import groups (stdlib, third-party, internal).
- Opening brace on same line as `func`/`if`/`for`.

## Naming

- Exported: `PascalCase`. Unexported: `camelCase`.
- Interfaces with one method: method name + `-er` suffix (`Reader`, `Stringer`).
- No stuttering: `http.Client`, not `http.HttpClient`.
- Acronyms all-caps: `HTTPServer`, `userID`.
- No `Get` prefix for getters: `Owner()`, `SetOwner()`.
- Receivers: 1-2 letter abbreviation, consistent. Never `this`/`self`.
- Errors: `var ErrNotFound = errors.New(...)`. Error types: suffix `Error`.

## Functions

- Small, one thing. Max ~40 lines.
- Max 3 params. Beyond that, use options struct or functional options pattern.
- Return `(Result, error)` — error always last.
- Use `defer` for cleanup (LIFO order).
- Named returns only when they improve documentation.

## Types & Interfaces

- No classes. Compose via structs, interfaces, embedding.
- Make the zero value useful.
- Define interfaces at the consumer, not implementor. Accept interfaces, return concrete types.
- Small interfaces (1-2 methods). Don't define preemptively.
- Embed for method promotion, not polymorphism. Use named field when promotion is unwanted.
- Pass typed structs between functions — not `map[string]interface{}` or raw strings.
- Define clear struct types for all data that crosses package boundaries.
- At system boundaries (API responses, config files, CLI args), unmarshal into typed structs immediately.
- Use interfaces for behavioural contracts, structs for data.

## Documentation

- Every exported name gets a doc comment starting with the name.
- `// FuncName does...` — complete sentences.
- Package comments in `doc.go` or primary file.
- Comments explain why, not what. No commented-out code.

## Error Handling

- Always check returned errors. Handle once, at the level with enough context.
- Wrap with `fmt.Errorf("context: %w", err)`. Use `%w` for wrappable, `%v` to hide.
- Sentinel errors: `var Err... = errors.New(...)`. Check with `errors.Is`/`errors.As`.
- No `panic` in library code. `recover` only at goroutine boundaries.

## Context

- `context.Context` always first parameter, named `ctx`.
- Never store context in a struct. Never pass `nil` — use `context.TODO()`.
- Always `defer cancel()` after `WithCancel`/`WithTimeout`/`WithDeadline`.
- Forward `ctx` to every downstream call. Never replace with `context.Background()` mid-chain.
- Context values: unexported key types, immutable values only. For trace IDs and auth claims, not config or DB pools.
- Use `*Context` variants of database/sql methods: `QueryContext`, `ExecContext`.

## Imports

- Group: stdlib, third-party, internal. Use `goimports`.
- One module per concern: one logger, one HTTP library, one JSON library.
- Use `internal/` for encapsulation.

## Concurrency

- Every goroutine needs a clear owner and shutdown mechanism (context, channel close).
- `sync.WaitGroup` for waiting. `errgroup.Group` for goroutines returning errors.
- Channels for ownership transfer and signaling. Mutexes for shared state.
- Run `go test -race` in CI. Zero tolerance for data races.

## Testing

- `_test.go` files alongside code. `TestXxx_Scenario_Outcome` naming.
- Table-driven tests. `t.Helper()` in test helpers.
- Mock via small interfaces. `httptest.NewServer` for HTTP tests.

## Database

- Explicit transactions with `db.BeginTx`. Always `defer tx.Rollback()`.
- Parameterised queries: `$1`, `$2` (or `?`) — never `fmt.Sprintf` or string concatenation.
- Use `db.Query(query, args...)` — never embed values in the query string.
- Validate and constrain input before it reaches the database layer.
- Configure pool: `SetMaxOpenConns`, `SetMaxIdleConns`, `SetConnMaxLifetime`.
- Always `defer rows.Close()` and check `rows.Err()`.

## Patterns

- Constructors: `NewXxx()` functions. Builder: functional options or builder struct.
- Singleton: `sync.Once`. Decorator: middleware wrapping `http.Handler`.
- Strategy: interface or function type. Template Method: interface + driver function.

## Defensive Programming

- Validate all external input at handler/service boundaries (request bodies, query params, config, env vars).
- Validate string lengths, numeric ranges, and patterns at the point of entry.
- Use strong types (custom types, enums via `iota`) to constrain domains.
- Check slice bounds before indexing. Validate pointers are non-nil before dereferencing from external sources.
- Sanitize input in file paths (`filepath.Clean`, `filepath.Rel`), URLs, or command execution.
- Never use `os/exec` with unsanitized user input. Use `strconv` for safe string-to-number conversion.

## Project Structure

- `cmd/` for binaries, `internal/` for private packages.
- Keep `main()` thin: parse flags, call `run()`, handle exit.
- One `go.mod` per deployable unit. Commit `go.sum`.

## Tooling

- `gofmt`/`goimports`, `golangci-lint`, `go vet`, `go test -race`, `govulncheck`.

## Build Tools

- `go build` is built-in; no external build tool needed.
- Use Makefiles for common targets (build, test, clean, lint).
- `go install` for installing tools to `$GOBIN`.
- Cross-compile: `GOOS=linux GOARCH=amd64 go build`.
- Multi-stage Docker builds for minimal images.
- `go get` for dependency version management.

## SBOM Creation

- Use `cyclonedx-gomod` for CycloneDX SBOM generation (JSON/XML). Note: `cyclonedx-go` is the library, `cyclonedx-gomod` is the CLI.
- Run `govulncheck` (official Go vulnerability tool) on every commit.
- Use `go-licenses` (Google) for license compliance verification — `mitchellh/golicense` is archived.
- `go.sum` provides cryptographic verification; include in VCS.
- Generate SBOM on release; store with binaries.
- Gate deployments on clean `govulncheck` and license compliance.
