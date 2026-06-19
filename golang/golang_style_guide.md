# Go Coding Style Guidelines

A comprehensive guide rooted in *Effective Go*, the Go Code Review Comments,
the standard library conventions, and established software engineering
literature — notably *Design Patterns* (Gamma et al., 1994) and *Clean Code*
(Martin, 2008).

---

## Table of Contents

1. [Philosophy](#1-philosophy)
2. [Code Layout & Formatting](#2-code-layout--formatting)
3. [Naming Conventions](#3-naming-conventions)
4. [Functions](#4-functions)
5. [Types, Interfaces & Composition](#5-types-interfaces--composition)
6. [Documentation & Comments](#6-documentation--comments)
7. [Error Handling](#7-error-handling)
8. [Packages & Imports](#8-packages--imports)
9. [Concurrency](#9-concurrency)
10. [Context](#10-context)
11. [Design Patterns in Go](#11-design-patterns-in-go)
12. [Testing](#12-testing)
13. [Database Access & ACID](#13-database-access--acid)
14. [Performance & Idiomatic Go](#14-performance--idiomatic-go)
15. [Defensive Programming & Input Validation](#15-defensive-programming--input-validation)
16. [Project Structure](#16-project-structure)
17. [Tooling](#17-tooling)
18. [Build Tools](#18-build-tools)
19. [SBOM Creation](#19-sbom-creation)
20. [References](#20-references)

---

## 1. Philosophy

### 1.1 Go Proverbs

Rob Pike's Go Proverbs capture the language's design ethos:

> Don't communicate by sharing memory, share memory by communicating.
> Concurrency is not parallelism.
> Channels orchestrate; mutexes serialize.
> The bigger the interface, the weaker the abstraction.
> Make the zero value useful.
> `interface{}` says nothing.
> Gofmt's style is no one's favourite, yet gofmt is everyone's favourite.
> A little copying is better than a little dependency.
> Syscall must always be guarded with build tags.
> Cgo must always be guarded with build tags.
> Cgo is not Go.
> With the unsafe package there are no guarantees.
> Clear is better than clever.
> Reflection is never clear.
> Errors are values.
> Don't just check errors, handle them gracefully.
> Design the architecture, name the components, document the details.
> Documentation is for users.

### 1.2 Guiding Principles

| Principle | Source | Summary |
|---|---|---|
| **Simplicity over cleverness** | *Effective Go*, Go Proverbs | Write straightforward code. If it needs a comment to explain *what* it does, rewrite it. |
| **Composition over inheritance** | Go language design, *Design Patterns* Ch. 1 | Go has no classes or inheritance. Compose behaviour through embedding and interfaces. |
| **Small interfaces** | Go Proverbs, *Effective Go* | The bigger the interface, the weaker the abstraction. Prefer one- or two-method interfaces. |
| **Explicit over implicit** | Go language design | No hidden control flow, no magic. Errors are returned, not thrown. |
| **Make the zero value useful** | Go Proverbs | A freshly declared variable should be usable without initialisation when possible. |
| **Single Responsibility** | *Clean Code* Ch. 10, SOLID | Every package, type, and function should have one reason to change. |
| **YAGNI** | XP / *Clean Code* | Do not build for hypothetical future requirements. |

---

## 2. Code Layout & Formatting

### 2.1 gofmt Is Non-Negotiable

All Go code **must** be formatted with `gofmt` (or `goimports`). There are no
style debates — the formatter is the authority.

- Use tabs for indentation (the `gofmt` default).
- Do not fight the formatter. If the output looks odd, reconsider the code
  structure rather than working around the tool.

### 2.2 Line Length

Go has no official line-length limit. However:

- Aim for **80–100 characters** as a soft guideline.
- Break long function signatures, struct literals, and chained calls for
  readability.
- Let `gofmt` handle alignment.

### 2.3 Blank Lines (*Effective Go*, `gofmt`)

`gofmt` does **not** add or remove blank lines — these are the programmer's
responsibility:

- **One blank line** between top-level function, type, and method
  definitions. No more than one.
- **No blank line** between a function signature and its opening brace
  (`gofmt` enforces brace on same line).
- **No blank line** at the start or end of a function body.
- **One blank line** between the `package` clause and the `import` block.
- **One blank line** between import groups (stdlib, third-party, internal).
- Use **single blank lines** within functions to separate logical sections.
  Two or more consecutive blank lines inside a function are never appropriate.
- Group related `const`, `var`, and `type` declarations. Separate unrelated
  groups with one blank line.

### 2.4 File Organisation

Within a `.go` file, order elements as follows:

1. `package` clause
2. `import` block
3. Constants (`const`)
4. Package-level variables (`var`)
5. Type declarations (`type`)
6. Constructor functions (`NewXxx`)
7. Methods (grouped by receiver type)
8. Free functions
9. `init()` (if absolutely necessary — prefer explicit initialisation)

---

## 3. Naming Conventions

Go's naming conventions are enforced by the language itself — exported names
start with an upper-case letter, unexported with lower-case.

### 3.1 General Rules (*Effective Go*, Go Code Review Comments)

| Entity | Convention | Example |
|---|---|---|
| Package | Short, lowercase, single word | `http`, `io`, `user` |
| Exported type | `PascalCase` | `UserService`, `Config` |
| Unexported type | `camelCase` | `requestContext` |
| Interface (1 method) | Method name + `-er` suffix | `Reader`, `Stringer`, `Closer` |
| Exported function | `PascalCase` | `NewServer()`, `ParseConfig()` |
| Unexported function | `camelCase` | `validateInput()` |
| Exported variable/const | `PascalCase` | `MaxRetries`, `ErrNotFound` |
| Unexported variable/const | `camelCase` | `defaultTimeout` |
| Error variable | `Err` prefix | `ErrNotFound`, `ErrTimeout` |
| Error type | `Error` suffix | `NotFoundError`, `ValidationError` |
| Receiver | Short (1–2 letters), consistent | `(s *Server)`, `(c *Client)` |

### 3.2 Naming Guidance (*Clean Code* Ch. 2, *Effective Go*)

- **Use short, clear names.** Go favours brevity — `srv` over `server` for
  local variables, `cfg` over `configuration`. But never sacrifice clarity:
  `userCount` beats `uc` at package scope.
- **No stuttering.** A type in package `http` should be `Client`, not
  `HttpClient` — it's accessed as `http.Client`.
- **Acronyms are all-caps.** `HTTPServer`, `userID`, `xmlParser` — not
  `HttpServer`, `userId`, `XmlParser`.
- **No `Get` prefix for getters.** A field `owner` has a getter `Owner()` and
  a setter `SetOwner()`.
- **Use intention-revealing names.** `elapsedTimeInDays` beats `d` at broader
  scope.
- **Avoid meaningless distinctions.** `fetchUser()` vs `retrieveUser()` in the
  same package — pick one style and stay consistent.

### 3.3 Receiver Names

Use a one- or two-letter abbreviation of the type name, applied consistently
across all methods:

```go
type Server struct { ... }

func (s *Server) Start() error   { ... }
func (s *Server) Stop() error    { ... }
func (s *Server) handler() http.Handler { ... }
```

Never use `this` or `self`.

---

## 4. Functions

### 4.1 Size and Focus (*Clean Code* Ch. 3)

- Functions should be **small** and do **one thing**.
- Extract nested logic into well-named helpers.
- If a function spans more than a screen (~40 lines), consider splitting it.

### 4.2 Parameters

- **Fewer parameters are better.** Group related parameters into a struct
  (options pattern) when a function needs more than three.
- Use the **functional options pattern** for flexible, extensible
  configuration:

```go
// Option configures a Server.
type Option func(*Server)

// WithTimeout sets the server's request timeout.
func WithTimeout(d time.Duration) Option {
	return func(s *Server) {
		s.timeout = d
	}
}

// WithLogger sets the server's logger.
func WithLogger(l *slog.Logger) Option {
	return func(s *Server) {
		s.logger = l
	}
}

// NewServer creates a Server with the given options.
func NewServer(addr string, opts ...Option) *Server {
	s := &Server{addr: addr, timeout: 30 * time.Second}
	for _, opt := range opts {
		opt(s)
	}
	return s
}
```

### 4.3 Return Values

- Return **errors as the last value**: `func Foo() (Result, error)`.
- Return **named results** only when it improves documentation or enables
  deferred error handling — not as a default.
- Prefer returning a **zero value plus an error** over returning a pointer that
  may be `nil`.

### 4.4 `defer` for Cleanup

Use `defer` to pair resource acquisition with release:

```go
// ReadFile reads and returns the entire contents of the named file.
func ReadFile(path string) ([]byte, error) {
	f, err := os.Open(path)
	if err != nil {
		return nil, fmt.Errorf("open %s: %w", path, err)
	}
	defer f.Close()

	return io.ReadAll(f)
}
```

- `defer` runs in LIFO order — use this for multi-resource cleanup.
- Be aware that `defer` captures arguments at the time of the `defer` call,
  not at execution.

---

## 5. Types, Interfaces & Composition

Go has no classes and no inheritance. Behaviour is built through **struct
types**, **interfaces**, and **embedding**.

### 5.1 Structs

- Group related data into structs.
- **Make the zero value useful** — design fields with sensible defaults:

```go
// Counter is safe to use without explicit initialisation.
// Its zero value is a counter starting at 0.
type Counter struct {
	mu sync.Mutex
	n  int
}

// Increment adds 1 to the counter.
func (c *Counter) Increment() {
	c.mu.Lock()
	defer c.mu.Unlock()
	c.n++
}
```

- Order fields from most to least important for readability. Consider
  alignment to reduce memory padding in performance-critical code.

### 5.2 Interfaces (*Effective Go*, Go Proverbs)

> "The bigger the interface, the weaker the abstraction."

- **Define interfaces at the consumer**, not the implementor. Accept
  interfaces, return concrete types.
- Prefer **small interfaces** — one or two methods:

```go
// Writer is the standard interface for writing bytes.
type Writer interface {
	Write(p []byte) (n int, err error)
}

// Storer persists domain objects.
type Storer interface {
	Store(ctx context.Context, key string, value []byte) error
}
```

- **Do not define interfaces preemptively.** Wait until you have two or more
  implementations or a clear need for decoupling (e.g. testing).
- Use the standard library interfaces (`io.Reader`, `io.Writer`, `fmt.Stringer`,
  `error`) wherever they fit.

### 5.3 Embedding (Composition, Not Inheritance)

Go provides embedding as its composition mechanism. An embedded type's methods
are promoted to the outer type:

```go
type Logger struct {
	*slog.Logger
}

type Application struct {
	Logger
	config Config
}
```

- **Embed for reuse, not for polymorphism.** Embedding is delegation, not
  subclassing.
- Be cautious — embedding exposes the inner type's exported methods. Only
  embed when you want those methods promoted.
- Prefer a named field when you want delegation without promotion:

```go
type Application struct {
	logger *slog.Logger  // use a.logger.Info(...), not a.Info(...)
	config Config
}
```

### 5.4 Type Assertions and Type Switches

Prefer type switches over chains of type assertions:

```go
// Describe returns a human-readable description of the shape's type.
func Describe(s Shape) string {
	switch v := s.(type) {
	case Circle:
		return fmt.Sprintf("circle with radius %.2f", v.Radius)
	case Rectangle:
		return fmt.Sprintf("%dx%d rectangle", v.Width, v.Height)
	default:
		return "unknown shape"
	}
}
```

### 5.5 Typed Data Passing

Pass **typed structs** between functions — not `map[string]interface{}`, raw
JSON strings, or unstructured data. Typed data makes interfaces explicit,
catches errors at compile time, and keeps code navigable.

- **Pass typed structs** between functions, not `map[string]interface{}` or raw
  strings. Define a struct for any data that crosses a function or package
  boundary:

```go
// CreateUserRequest holds validated input for user creation.
type CreateUserRequest struct {
	Email       string
	DisplayName string
}

// Yes — typed struct
func CreateUser(ctx context.Context, req CreateUserRequest) (*User, error) { ... }

// No — unstructured map
func CreateUser(ctx context.Context, data map[string]interface{}) (*User, error) { ... }
```

- **Define clear struct types** for all data that crosses package boundaries.
  Exported structs serve as the contract between packages.
- **At system boundaries** (API responses, config files, CLI arguments),
  **unmarshal into typed structs immediately**. Downstream code should never
  operate on raw `[]byte` or `map[string]interface{}`:

```go
var cfg Config
if err := json.Unmarshal(data, &cfg); err != nil {
	return fmt.Errorf("parse config: %w", err)
}
// use cfg (typed) from here on
```

- **Use interfaces for behavioural contracts, structs for data.** An interface
  says "what can you do?"; a struct says "what data do you carry?"

### 5.6 SOLID in Go

| Principle | Go Application |
|---|---|
| **S** — Single Responsibility | One package per concern; one struct per concept. |
| **O** — Open/Closed | Extend via new interface implementations, embedding, or functional options — not by editing existing code. |
| **L** — Liskov Substitution | Any type satisfying an interface must honour its contract. Go's implicit interfaces encourage this naturally. |
| **I** — Interface Segregation | Keep interfaces small. Compose larger interfaces from smaller ones (`io.ReadWriter = io.Reader + io.Writer`). |
| **D** — Dependency Inversion | Accept interfaces in function parameters, return concrete types. Inject dependencies through constructors. |

---

## 6. Documentation & Comments

### 6.1 Doc Comments (*Effective Go*, Go Code Review Comments)

Every exported name **must** have a doc comment. Doc comments are complete
sentences that begin with the name of the element:

```go
// Config holds the application configuration.
//
// Zero-value Config uses sensible defaults for all fields.
type Config struct {
	// Addr is the TCP address to listen on (default ":8080").
	Addr string

	// ReadTimeout is the maximum duration for reading a request.
	ReadTimeout time.Duration
}

// NewConfig returns a Config populated from environment variables.
//
// It returns an error if a required variable is missing or malformed.
func NewConfig() (*Config, error) {
	...
}
```

- **Start with the name.** `// NewConfig returns ...` not `// This function
  creates ...`.
- Use a **blank `//` line** to separate paragraphs within a doc comment.
- Document **why**, not what, when the code is self-explanatory.
- Package comments go in a file named `doc.go` or at the top of the primary
  file:

```go
// Package auth provides authentication and authorisation middleware
// for the HTTP layer.
package auth
```

### 6.2 Comments on Unexported Code

Unexported functions and types do not require doc comments if the name and
signature are clear. Add comments when:

- The implementation has non-obvious behaviour.
- A workaround exists for an external bug.
- A performance trade-off is intentional.

### 6.3 Comment Hygiene (*Clean Code* Ch. 4)

- Never commit commented-out code — version control preserves history.
- Delete redundant comments that restate the code.
- Do not use comments as section banners (`// ====== HELPERS ======`). File
  organisation and package boundaries do that job.

---

## 7. Error Handling

### 7.1 Principles (Go Proverbs, *Effective Go*)

> "Errors are values."
> "Don't just check errors, handle them gracefully."

- **Always check returned errors.** Ignoring an error is a conscious decision
  and must be explicitly marked with a comment.
- **Handle errors once** — at the level that has enough context to act on them.
- **Wrap errors** with `fmt.Errorf("context: %w", err)` to preserve the error
  chain.

### 7.2 Error Wrapping (Go 1.13+)

```go
// LoadConfig reads and parses the configuration file.
func LoadConfig(path string) (*Config, error) {
	data, err := os.ReadFile(path)
	if err != nil {
		return nil, fmt.Errorf("load config from %s: %w", path, err)
	}

	var cfg Config
	if err := json.Unmarshal(data, &cfg); err != nil {
		return nil, fmt.Errorf("parse config: %w", err)
	}
	return &cfg, nil
}
```

- Use `%w` to wrap so callers can use `errors.Is` and `errors.As`.
- Use `%v` when you deliberately want to **hide** the underlying error from
  callers.

### 7.3 Sentinel Errors and Custom Types

```go
// Package-level sentinel errors.
var (
	ErrNotFound    = errors.New("not found")
	ErrUnauthorized = errors.New("unauthorized")
)

// ValidationError carries field-level details.
type ValidationError struct {
	Field   string
	Message string
}

func (e *ValidationError) Error() string {
	return fmt.Sprintf("validation: %s — %s", e.Field, e.Message)
}
```

- Use **sentinel errors** (`var Err... = errors.New(...)`) for well-known,
  constant conditions.
- Use **custom error types** when callers need to inspect structured data.
- Check with `errors.Is` (sentinels) and `errors.As` (types), never `==`.

### 7.4 Panic and Recover

- **Do not panic** in library code. Panics are for truly unrecoverable
  situations (programmer errors, impossible states).
- If you must panic in `init()` or in a test helper, document it.
- Use `recover` only at the top of a goroutine boundary (e.g. an HTTP handler)
  to prevent one request from crashing the server.

---

## 8. Packages & Imports

### 8.1 Package Design (*Effective Go*)

- A package provides **one idea** — `http`, `json`, `auth`.
- Package names are short, lowercase, single-word nouns. No `util`, `common`,
  `misc` — they are symptoms of unclear boundaries.
- Avoid package-level state. If you need it, protect it with a mutex and
  document the concurrency contract.

### 8.2 Import Ordering

Group imports separated by blank lines:

1. **Standard library**
2. **Third-party**
3. **Internal / project**

```go
import (
	"context"
	"fmt"
	"net/http"

	"github.com/go-chi/chi/v5"
	"go.uber.org/zap"

	"mycompany/myapp/internal/auth"
	"mycompany/myapp/internal/store"
)
```

Use `goimports` to automate ordering and prune unused imports.

### 8.3 One Module per Concern — No Redundant Imports

When multiple packages can accomplish the same task, **pick one and use it
consistently** across the file and project:

```go
// Bad — mixing packages for the same purpose
import (
	"log"
	"go.uber.org/zap"
)

// Good — pick one
import "go.uber.org/zap"
```

Common violations:

| Overlapping packages | Pick one |
|---|---|
| `log` / `slog` / `zap` / `zerolog` | One structured logger project-wide |
| `encoding/json` / `github.com/json-iterator/go` | One JSON library per module |
| `net/http` / `fasthttp` | One HTTP server implementation |
| `sync.Mutex` / channel for simple locking | `sync.Mutex` for state protection, channels for communication |
| `os.Getenv` / `viper` / `envconfig` for config | One config approach per project |
| `fmt.Sprintf` / `strconv` for number→string | `strconv` for primitives, `fmt` for complex formatting |

### 8.4 Internal Packages

Use Go's `internal/` convention to enforce encapsulation boundaries:

```
myapp/
    internal/       # only importable by myapp and its children
        store/
        auth/
    cmd/
        server/
```

---

## 9. Concurrency

### 9.1 Principles (Go Proverbs, *Effective Go*)

> "Don't communicate by sharing memory, share memory by communicating."
> "Concurrency is not parallelism."

- Goroutines are cheap but not free. Always ensure every goroutine has a clear
  exit path.
- Use `context.Context` for cancellation, deadlines, and request-scoped values.

### 9.2 Goroutine Lifecycle

Every goroutine must have a **clear owner** and a **shutdown mechanism**:

```go
// Worker processes items from the queue until ctx is cancelled.
func (s *Service) Worker(ctx context.Context, queue <-chan Job) {
	for {
		select {
		case <-ctx.Done():
			return
		case job, ok := <-queue:
			if !ok {
				return
			}
			s.process(job)
		}
	}
}
```

- Use `sync.WaitGroup` to wait for goroutines to finish.
- Use `errgroup.Group` (`golang.org/x/sync/errgroup`) when goroutines return
  errors.
- Never fire-and-forget a goroutine without a way to observe its completion or
  failure.

### 9.3 Channels vs. Mutexes

| Use case | Mechanism |
|---|---|
| Passing ownership of data | Channel |
| Signalling events (done, ready) | Channel or `context.Context` |
| Protecting shared state | `sync.Mutex` / `sync.RWMutex` |
| One-time initialisation | `sync.Once` |
| Counting concurrent work | `sync.WaitGroup` |

### 9.4 Common Pitfalls

- **Goroutine leak.** Every goroutine must have a termination condition —
  closing a channel, cancelling a context, or both.
- **Race conditions.** Run `go test -race ./...` in CI. Zero tolerance for
  data races.
- **Channel deadlock.** Unbuffered channels block both sender and receiver.
  Size buffers intentionally, not to suppress symptoms.
- **Closure capture.** When launching goroutines in a loop, capture the loop
  variable explicitly (Go <1.22) or rely on per-iteration scoping (Go 1.22+).

---

## 10. Context

The `context` package is central to Go's approach to cancellation, deadlines,
and request-scoped data. Every function that performs I/O, calls a downstream
service, or may run for a non-trivial duration **must** accept a
`context.Context` as its first parameter.

### 10.1 Core Rules

- **`context.Context` is always the first parameter**, named `ctx`:

```go
func FetchUser(ctx context.Context, id int64) (*User, error) { ... }
```

- **Never store a context** in a struct. Pass it explicitly through the call
  chain. A context is scoped to a single request or operation — storing it
  couples object lifetime to request lifetime.

```go
// Bad — context outlives the request
type Service struct {
	ctx context.Context  // never do this
}

// Good — pass per call
func (s *Service) Process(ctx context.Context, job Job) error { ... }
```

- **Never pass `nil`** for a context. If unsure which context to use, pass
  `context.TODO()` and leave a comment explaining why. Use
  `context.Background()` only at the top of `main()`, `init()`, and test
  setup.

### 10.2 Cancellation

Use `context.WithCancel` when the caller needs to signal that work is no
longer needed:

```go
// MonitorHealth checks the service and cancels dependent work on failure.
func MonitorHealth(ctx context.Context, svc *Service) error {
	ctx, cancel := context.WithCancel(ctx)

	var wg sync.WaitGroup
	wg.Add(1)
	go func() {
		defer wg.Done()
		svc.Worker(ctx)
	}()

	// Order matters: cancel() must run BEFORE wg.Wait() so the worker
	// observes cancellation and exits. defer is LIFO, so defer Wait first.
	defer wg.Wait()
	defer cancel()

	if err := svc.HealthCheck(ctx); err != nil {
		return fmt.Errorf("health check: %w", err)
	}
	return nil
}
```

- **Always `defer cancel()`** immediately after creating a cancellable context.
  This ensures the context is released even on early returns and prevents
  resource leaks.
- Calling `cancel()` more than once is safe and idiomatic — the deferred call
  is a safety net, not a conflict.

### 10.3 Deadlines and Timeouts

Use `context.WithTimeout` or `context.WithDeadline` to enforce time bounds on
operations:

```go
// FetchWithTimeout retrieves data with a hard time limit.
func FetchWithTimeout(ctx context.Context, url string) ([]byte, error) {
	ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
	defer cancel()

	req, err := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
	if err != nil {
		return nil, fmt.Errorf("create request: %w", err)
	}

	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return nil, fmt.Errorf("fetch %s: %w", url, err)
	}
	defer resp.Body.Close()

	return io.ReadAll(resp.Body)
}
```

- A child timeout **cannot** extend a parent's deadline. If the parent has 3s
  remaining, `WithTimeout(ctx, 10*time.Second)` still expires in 3s.
- Check `ctx.Err()` to distinguish `context.Canceled` (caller gave up) from
  `context.DeadlineExceeded` (ran out of time).

### 10.4 Context Values

`context.WithValue` attaches request-scoped metadata (trace IDs, auth tokens,
request IDs) to the context. **Use it sparingly** — it bypasses the type system
and makes dependencies invisible.

```go
// Define an unexported key type to prevent collisions.
type contextKey struct{}

var requestIDKey = contextKey{}

// WithRequestID returns a context carrying the given request ID.
func WithRequestID(ctx context.Context, id string) context.Context {
	return context.WithValue(ctx, requestIDKey, id)
}

// RequestID extracts the request ID from the context, or returns "" if absent.
func RequestID(ctx context.Context) string {
	id, _ := ctx.Value(requestIDKey).(string)
	return id
}
```

Rules for context values:

- **Use an unexported type as the key** to prevent other packages from
  colliding with your keys.
- **Never use context values for optional function parameters.** If a function
  needs data to do its job, accept it as an explicit argument.
- **Never store mutable state** in context values. Values should be immutable
  and safe for concurrent reads.
- Valid uses: trace/request IDs, authentication claims, logger instances.
  Invalid uses: database connections, configuration, business logic inputs.

### 10.5 Propagation Through the Call Chain

Context flows **downward** through every layer — HTTP handlers, service
methods, repository calls, and external client calls:

```go
func (h *Handler) GetUser(w http.ResponseWriter, r *http.Request) {
	ctx := r.Context() // inherit the request context

	user, err := h.service.FindUser(ctx, userID)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}
	if err := json.NewEncoder(w).Encode(user); err != nil {
		// Headers already written; log and move on.
		slog.ErrorContext(ctx, "encode response", "err", err)
	}
}

func (s *UserService) FindUser(ctx context.Context, id int64) (*User, error) {
	return s.repo.GetByID(ctx, id) // pass ctx to the data layer
}

func (r *UserRepo) GetByID(ctx context.Context, id int64) (*User, error) {
	row := r.db.QueryRowContext(ctx, "SELECT id, name FROM users WHERE id = $1", id)
	// ...
}
```

- **Never drop the context.** If you receive a `ctx`, pass it to every
  downstream call that accepts one. Replacing it with `context.Background()`
  breaks cancellation and deadline propagation.
- Use `http.NewRequestWithContext` (not `http.NewRequest`) when making outbound
  HTTP calls.
- Use the `Context`-suffixed `database/sql` methods (`QueryContext`,
  `ExecContext`, `QueryRowContext`) — the non-context variants ignore
  cancellation.

### 10.6 Context in Tests

In tests, create a context that automatically cancels when the test ends:

```go
func TestFetchUser(t *testing.T) {
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()

	user, err := service.FetchUser(ctx, 42)
	if err != nil {
		t.Fatalf("FetchUser: %v", err)
	}
	// ...
}
```

For tests that need a cancelled context:

```go
func TestFetchUser_Cancelled(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel() // satisfies the lostcancel vet check
	cancel()       // immediately cancel before the call

	_, err := service.FetchUser(ctx, 42)
	if !errors.Is(err, context.Canceled) {
		t.Errorf("expected context.Canceled, got %v", err)
	}
}
```

### 10.7 Common Mistakes

| Mistake | Why it's wrong | Fix |
|---|---|---|
| Storing `ctx` in a struct field | Couples object lifetime to request lifetime; stale context | Pass `ctx` as a function parameter |
| Passing `context.Background()` mid-chain | Breaks parent cancellation and deadlines | Forward the received `ctx` |
| Using `string` as context key | Key collisions between packages | Use an unexported `struct{}` type |
| Putting config or DB pools in context values | Hides real dependencies, untestable | Use explicit constructor parameters |
| Forgetting `defer cancel()` | Context goroutine leaks until parent is cancelled | Always defer immediately after creation |
| Ignoring `ctx.Done()` in long loops | Operation cannot be cancelled | Check `ctx.Done()` in `select` |

---

## 11. Design Patterns in Go

The GoF patterns remain relevant, but Go's composition model, first-class
functions, and interfaces replace much of the class-based machinery.

### 11.1 Creational Patterns

#### Constructor Functions

Go has no constructors. Use `NewXxx` functions:

```go
// NewClient creates an HTTP client with the given base URL.
func NewClient(baseURL string, opts ...Option) *Client {
	c := &Client{baseURL: baseURL, httpClient: http.DefaultClient}
	for _, opt := range opts {
		opt(c)
	}
	return c
}
```

#### Builder

Use the functional options pattern (section 4.2) or a dedicated builder struct
for complex construction:

```go
// QueryBuilder constructs SQL SELECT statements.
type QueryBuilder struct {
	table      string
	conditions []string
	limit      int
}

// Where adds a WHERE condition.
func (qb *QueryBuilder) Where(cond string) *QueryBuilder {
	qb.conditions = append(qb.conditions, cond)
	return qb
}

// Limit sets the maximum number of rows.
func (qb *QueryBuilder) Limit(n int) *QueryBuilder {
	qb.limit = n
	return qb
}

// Build renders the final SQL string.
func (qb *QueryBuilder) Build() string {
	sql := fmt.Sprintf("SELECT * FROM %s", qb.table)
	if len(qb.conditions) > 0 {
		sql += " WHERE " + strings.Join(qb.conditions, " AND ")
	}
	if qb.limit > 0 {
		sql += fmt.Sprintf(" LIMIT %d", qb.limit)
	}
	return sql
}
```

#### Singleton

Module-level variables or `sync.Once` replace the Singleton class:

```go
var (
	registryOnce sync.Once
	registry     *Registry
)

// GetRegistry returns the process-wide registry, creating it on first call.
func GetRegistry() *Registry {
	registryOnce.Do(func() {
		registry = &Registry{}
	})
	return registry
}
```

### 11.2 Structural Patterns

#### Decorator (Middleware)

Go's `http.Handler` interface is a textbook Decorator:

```go
// WithLogging wraps an http.Handler with request/response logging.
func WithLogging(next http.Handler, logger *slog.Logger) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		next.ServeHTTP(w, r)
		logger.Info("request",
			"method", r.Method,
			"path", r.URL.Path,
			"duration", time.Since(start),
		)
	})
}
```

#### Adapter

Implement a target interface by wrapping an incompatible type:

```go
// LegacyPrinter has a non-standard API.
type LegacyPrinter struct{}

// PrintOld is the legacy printing method.
func (lp *LegacyPrinter) PrintOld(text string) { ... }

// Printer is the standard interface.
type Printer interface {
	Print(text string)
}

// LegacyAdapter adapts LegacyPrinter to the Printer interface.
type LegacyAdapter struct {
	legacy *LegacyPrinter
}

// Print delegates to the legacy printer's PrintOld method.
func (a *LegacyAdapter) Print(text string) {
	a.legacy.PrintOld(text)
}
```

### 11.3 Behavioural Patterns

#### Strategy

Pass behaviour as an interface or a function value:

```go
// Compressor compresses raw data.
type Compressor interface {
	Compress(data []byte) ([]byte, error)
}

// Archiver archives payloads using a pluggable compression strategy.
type Archiver struct {
	compressor Compressor
}

// Archive compresses the payload.
func (a *Archiver) Archive(payload []byte) ([]byte, error) {
	return a.compressor.Compress(payload)
}
```

Or simply use a function type for one-method strategies:

```go
// CompressFunc is an adapter to allow the use of ordinary functions
// as Compressors.
type CompressFunc func([]byte) ([]byte, error)

// Compress calls f(data).
func (f CompressFunc) Compress(data []byte) ([]byte, error) {
	return f(data)
}
```

#### Observer / Event System

Use channels or callback slices:

```go
// EventEmitter provides a simple publish-subscribe mechanism.
type EventEmitter struct {
	mu        sync.RWMutex
	listeners map[string][]func(any)
}

// NewEventEmitter returns an EventEmitter ready for use.
func NewEventEmitter() *EventEmitter {
	return &EventEmitter{listeners: make(map[string][]func(any))}
}

// On registers a callback for the given event.
func (e *EventEmitter) On(event string, cb func(any)) {
	e.mu.Lock()
	defer e.mu.Unlock()
	e.listeners[event] = append(e.listeners[event], cb)
}

// Emit fires an event, calling all registered listeners.
//
// Callbacks run *after* the lock is released so a callback can safely call
// On/Emit without deadlocking and a slow callback does not block other
// goroutines from registering listeners.
func (e *EventEmitter) Emit(event string, data any) {
	e.mu.RLock()
	cbs := make([]func(any), len(e.listeners[event]))
	copy(cbs, e.listeners[event])
	e.mu.RUnlock()
	for _, cb := range cbs {
		cb(data)
	}
}
```

#### Template Method

Use an interface plus a driver function — Go has no abstract methods, so the
caller supplies the varying steps:

```go
// Pipeline defines the steps for an ETL process.
type Pipeline interface {
	Extract(source string) ([]byte, error)
	Transform(raw []byte) (map[string]any, error)
	Load(data map[string]any) error
}

// RunPipeline executes the full extract-transform-load cycle.
func RunPipeline(p Pipeline, source string) error {
	raw, err := p.Extract(source)
	if err != nil {
		return fmt.Errorf("extract: %w", err)
	}
	data, err := p.Transform(raw)
	if err != nil {
		return fmt.Errorf("transform: %w", err)
	}
	return p.Load(data)
}
```

---

## 12. Testing

### 12.1 Principles

- Tests live **alongside the code** they test, in `_test.go` files.
- Use the `testing` package. Avoid test frameworks unless the project has
  agreed on one (e.g. `testify` for assertions).
- **Test behaviour, not implementation.** Tests must survive refactors that
  preserve behaviour.
- Follow the **testing pyramid**: unit tests > integration tests > e2e tests.

### 12.2 Naming

Test functions follow `TestXxx` and describe the **scenario and outcome**:

```go
func TestWithdraw_InsufficientFunds_ReturnsError(t *testing.T) { ... }
func TestParseCSV_EmptyFile_ReturnsEmptySlice(t *testing.T) { ... }
```

### 12.3 Table-Driven Tests

The dominant Go testing idiom — each case is explicit and self-documenting:

```go
func TestAdd(t *testing.T) {
	tests := []struct {
		name string
		a, b int
		want int
	}{
		{name: "positives", a: 2, b: 3, want: 5},
		{name: "zero", a: 0, b: 0, want: 0},
		{name: "negatives", a: -1, b: -2, want: -3},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			got := Add(tt.a, tt.b)
			if got != tt.want {
				t.Errorf("Add(%d, %d) = %d, want %d", tt.a, tt.b, got, tt.want)
			}
		})
	}
}
```

### 12.4 Test Helpers

Use `t.Helper()` in helper functions so failures report the caller's line:

```go
// mustParseURL parses a URL or fails the test.
func mustParseURL(t *testing.T, raw string) *url.URL {
	t.Helper()
	u, err := url.Parse(raw)
	if err != nil {
		t.Fatalf("parse URL %q: %v", raw, err)
	}
	return u
}
```

### 12.5 Mocking

- Prefer **small interfaces** and pass them as dependencies. Tests supply a
  stub implementation.
- Only mock at **system boundaries** — HTTP clients, databases, clocks.
- Use `httptest.NewServer` for HTTP integration tests.

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
| **Isolation** | Concurrent transactions do not interfere with each other. | Two goroutines read the same balance, both withdraw, and the final balance is wrong. |
| **Durability** | Once committed, data survives crashes and power loss. | A commit returns successfully but the data is lost after a restart. |

### 13.2 Always Use Explicit Transactions

Never rely on auto-commit for multi-statement operations. Use `db.BeginTx`
to wrap related writes in a single transaction:

```go
// Transfer moves funds between two accounts atomically.
func Transfer(ctx context.Context, db *sql.DB, fromID, toID int64, amount float64) error {
	tx, err := db.BeginTx(ctx, nil)
	if err != nil {
		return fmt.Errorf("begin tx: %w", err)
	}
	defer tx.Rollback() // no-op after Commit

	if _, err := tx.ExecContext(ctx,
		"UPDATE accounts SET balance = balance - $1 WHERE id = $2",
		amount, fromID,
	); err != nil {
		return fmt.Errorf("debit account %d: %w", fromID, err)
	}

	if _, err := tx.ExecContext(ctx,
		"UPDATE accounts SET balance = balance + $1 WHERE id = $2",
		amount, toID,
	); err != nil {
		return fmt.Errorf("credit account %d: %w", toID, err)
	}

	if err := tx.Commit(); err != nil {
		return fmt.Errorf("commit transfer: %w", err)
	}
	return nil
}
```

- Always `defer tx.Rollback()` immediately after `BeginTx`. It is a no-op
  after a successful `Commit()`, but ensures cleanup on error paths.
- Pass `context.Context` through all database calls — it carries deadlines and
  cancellation signals.

### 13.3 Choose the Correct Isolation Level

Most databases default to READ COMMITTED. Set a stricter level when the
operation demands it:

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Use When |
|---|---|---|---|---|
| READ UNCOMMITTED | Possible | Possible | Possible | Almost never — analytics on stale data at most |
| READ COMMITTED | No | Possible | Possible | Default for most OLTP workloads |
| REPEATABLE READ | No | No | Possible | Reports or computations that must see a stable snapshot |
| SERIALIZABLE | No | No | No | Financial transactions, inventory, anything where correctness is paramount |

```go
tx, err := db.BeginTx(ctx, &sql.TxOptions{
	Isolation: sql.LevelSerializable,
})
```

### 13.4 Use Parameterised Queries — Never String Interpolation

This protects both **consistency** (correct types) and **security** (SQL
injection):

```go
// Yes — parameterised
row := db.QueryRowContext(ctx, "SELECT name FROM users WHERE id = $1", userID)

// No — string interpolation (SQL injection risk)
row := db.QueryRowContext(ctx, fmt.Sprintf("SELECT name FROM users WHERE id = %d", userID))
```

### 13.5 SQL Injection Protection

SQL injection is one of the most dangerous and prevalent vulnerabilities.
Every database interaction **must** be guarded against it.

- **Always use parameterised queries** with `$1`, `$2` (PostgreSQL) or `?`
  (MySQL/SQLite) placeholders via `database/sql`.
- **Never use `fmt.Sprintf`** or string concatenation to build SQL statements.
  Even when the value appears safe, string formatting bypasses the driver's
  parameter binding:

```go
// Bad — SQL injection via string formatting
query := fmt.Sprintf("SELECT * FROM users WHERE name = '%s'", name)
db.QueryContext(ctx, query)

// Good — parameterised
db.QueryContext(ctx, "SELECT * FROM users WHERE name = $1", name)
```

- **Use `db.Query(query, args...)`** — never embed values in the query string.
  The `database/sql` package transmits parameters out-of-band, so the database
  never interprets them as SQL.
- **Validate and constrain input** before it reaches the database layer.
  Restrict string lengths, check numeric ranges, and reject unexpected
  characters at the handler or service boundary.

### 13.6 Handle Connection Lifecycle Properly

- **Use `database/sql` connection pooling.** Configure `SetMaxOpenConns`,
  `SetMaxIdleConns`, and `SetConnMaxLifetime` based on workload.
- **Always close `*sql.Rows`** — use `defer rows.Close()` immediately after
  `Query`.
- **Check `rows.Err()`** after the scan loop to catch iteration errors.
- **Retry on transient errors** (connection reset, serialisation failure) with
  backoff, but never silently swallow the error.

```go
// ScanUsers reads all users from the query result.
func ScanUsers(ctx context.Context, db *sql.DB) ([]User, error) {
	rows, err := db.QueryContext(ctx, "SELECT id, name FROM users")
	if err != nil {
		return nil, fmt.Errorf("query users: %w", err)
	}
	defer rows.Close()

	var users []User
	for rows.Next() {
		var u User
		if err := rows.Scan(&u.ID, &u.Name); err != nil {
			return nil, fmt.Errorf("scan user: %w", err)
		}
		users = append(users, u)
	}
	if err := rows.Err(); err != nil {
		return nil, fmt.Errorf("iterate users: %w", err)
	}
	return users, nil
}
```

### 13.7 ORMs and Query Builders

When using an ORM or query builder (`sqlc`, `GORM`, `sqlx`), the same ACID
principles apply:

- Prefer **`sqlc`** (generates type-safe Go from SQL) — it keeps SQL explicit
  and parameterised.
- If using GORM, always use its `Transaction` method, never raw `db.Exec` for
  multi-statement writes.
- Use `SELECT ... FOR UPDATE` to prevent concurrent modifications when reading
  values that will be updated within the same transaction.
- Keep transactions **short** — hold locks for the minimum time necessary.

---

## 14. Performance & Idiomatic Go

### 14.1 Slices and Maps

```go
// Pre-allocate when the size is known.
users := make([]User, 0, len(ids))

// Use maps for lookups, not linear scans.
seen := make(map[string]bool, len(items))
```

### 14.2 String Building

Use `strings.Builder` for repeated concatenation:

```go
// Join builds a comma-separated string from the given items.
func Join(items []string) string {
	var b strings.Builder
	for i, item := range items {
		if i > 0 {
			b.WriteByte(',')
		}
		b.WriteString(item)
	}
	return b.String()
}
```

### 14.3 Avoid Premature Optimisation

- **Benchmark first** (`go test -bench=.`), then optimise.
- Use `pprof` for CPU and memory profiling.
- Optimise hot paths, not all paths.

### 14.4 Value vs. Pointer Receivers

| Use a **value receiver** when | Use a **pointer receiver** when |
|---|---|
| The struct is small and cheap to copy | The method modifies the receiver |
| The method does not mutate state | The struct is large |
| You want the type to be usable as a map key | Consistency — if one method needs a pointer, all should use pointers |

### 14.5 The `main` Function

Every executable Go program must have a `package main` with a `main()`
function. Structure it to keep logic testable:

```go
package main

import (
	"context"
	"fmt"
	"os"
)

func main() {
	if err := run(context.Background(), os.Args[1:]); err != nil {
		fmt.Fprintf(os.Stderr, "error: %v\n", err)
		os.Exit(1)
	}
}

// run contains the actual application logic, testable without os.Exit.
func run(ctx context.Context, args []string) error {
	...
}
```

- Keep `main()` thin — parse flags, call `run()`, handle the exit code.
- This pattern makes the core logic testable without `os.Exit` interference.

---

## 15. Defensive Programming & Input Validation

External input is untrusted by definition. Validate it at the point of entry so
that downstream code can operate on known-good data. Go's strong typing helps,
but types alone do not enforce constraints like "port in range 1–65535" or
"string shorter than 256 bytes."

### 15.1 Validate at Handler and Service Boundaries

- **Validate all external input** at HTTP handler or service method boundaries
  — request bodies, query parameters, headers, config values, CLI flags, and
  environment variables.
- **Validate string lengths, numeric ranges, and patterns** at the point of
  entry. Return a descriptive error immediately if validation fails:

```go
func (h *Handler) CreateUser(w http.ResponseWriter, r *http.Request) {
	var req CreateUserRequest
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		http.Error(w, "invalid request body", http.StatusBadRequest)
		return
	}
	if len(req.Name) == 0 || len(req.Name) > 255 {
		http.Error(w, "name must be 1-255 characters", http.StatusBadRequest)
		return
	}
	if req.Port < 1 || req.Port > 65535 {
		http.Error(w, "port must be 1-65535", http.StatusBadRequest)
		return
	}
	...
}
```

### 15.2 Use Strong Types to Constrain Domains

- Use **custom types** and **enums via `iota`** to restrict values at the type
  level:

```go
type Role int

const (
	RoleUser  Role = iota
	RoleAdmin
	RoleModerator
)
```

### 15.3 Defensive Access Patterns

- **Check slice bounds** before indexing. Out-of-bounds access panics at
  runtime in Go — validate lengths first.
- **Validate pointers are non-nil** before dereferencing when they come from
  external sources (decoded JSON, optional config fields, interface assertions).

### 15.4 Input Sanitisation

- **Sanitize input used in file paths** with `filepath.Clean` and
  `filepath.Rel` to prevent directory traversal:

```go
clean := filepath.Clean(userInput)
if !strings.HasPrefix(filepath.Join(baseDir, clean), baseDir) {
	return fmt.Errorf("path traversal detected: %s", userInput)
}
```

- **Never use `os/exec`** with unsanitized user input. Pass arguments as
  separate strings to `exec.Command`, never as part of a shell command string.
- **Use `strconv`** for safe string-to-number conversion. It returns errors
  for malformed input rather than producing silent zero values.

---

## 16. Project Structure

### 16.1 Recommended Layout

```
myapp/
    go.mod
    go.sum
    main.go                  # or cmd/myapp/main.go for multi-binary repos
    cmd/
        myapp/
            main.go          # thin main, calls internal packages
        worker/
            main.go
    internal/                # private to this module
        auth/
            auth.go
            auth_test.go
        store/
            store.go
            store_test.go
    pkg/                     # public API (use sparingly)
        client/
            client.go
    doc.go                   # package-level documentation
    Makefile
    Dockerfile
```

- Use `cmd/` for multiple binaries in one module.
- Use `internal/` to enforce encapsulation at the Go compiler level.
- Use `pkg/` sparingly — only for code explicitly intended as a public library.

### 16.2 Module Management

- One `go.mod` per deployable unit (service, CLI tool).
- **Run `go mod tidy`** after every dependency change.
- Commit `go.sum` to version control.
- Review dependency updates for security and compatibility regularly.

---

## 17. Tooling

### 17.1 Recommended Tool Chain

| Purpose | Tool | Notes |
|---|---|---|
| Formatter | `gofmt` / `goimports` | Non-negotiable — run on save |
| Linter | `golangci-lint` | Aggregates dozens of linters in one run |
| Static analysis | `go vet` | Built-in, catches common mistakes |
| Race detector | `go test -race` | Run in CI on every PR |
| Security scanner | `govulncheck` | Check for known vulnerabilities in deps |
| Test runner | `go test ./...` | With `-cover` for coverage |
| Benchmarking | `go test -bench=.` | With `-benchmem` for allocation stats |
| Profiling | `pprof` | CPU, memory, goroutine profiling |

### 17.2 CI Checks

At minimum, CI should run:

```bash
gofmt -l .               # formatting
go vet ./...              # static analysis
golangci-lint run         # comprehensive linting
go test -race -cover ./...  # tests with race detection
govulncheck ./...         # vulnerability scanning
```

---

## 18. Build Tools

### 18.1 go build (Native)

The Go toolchain is built-in; no external build tool needed:

```bash
go build ./...  # Build all packages
go build -o myapp ./cmd/myapp  # Build a package (directory), output named "myapp"
go build -ldflags "-X main.Version=1.0.0" ./...  # Set build variables
```

### 18.2 Makefiles for Go Projects

Common Makefile targets:

```makefile
.PHONY: build test clean

build:
	go build -o bin/myapp ./cmd/myapp

test:
	go test -v ./...

clean:
	rm -rf bin/

fmt:
	gofmt -s -w .
	go vet ./...

lint:
	golangci-lint run
```

### 18.3 go install for Tools

Install Go tools to `$GOBIN`:
```bash
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
go install golang.org/x/tools/cmd/goimports@latest
```

### 18.4 Multi-platform Builds

Build for different OS/architecture:
```bash
GOOS=linux GOARCH=amd64 go build -o bin/myapp-linux ./cmd/myapp
GOOS=darwin GOARCH=amd64 go build -o bin/myapp-macos ./cmd/myapp
GOOS=windows GOARCH=amd64 go build -o bin/myapp.exe ./cmd/myapp
```

### 18.5 Docker Builds

Multi-stage Docker builds for minimal image size:
```dockerfile
FROM golang:1.23 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /out/myapp ./cmd/myapp

FROM gcr.io/distroless/static-debian12:nonroot
COPY --from=builder /out/myapp /app/myapp
USER nonroot:nonroot
ENTRYPOINT ["/app/myapp"]
```

### 18.6 go.mod Versioning

Semantic versioning in imports:
```bash
go get github.com/some/lib@v1.2.3  # Specific version
go get github.com/some/lib@latest   # Latest
go get -u ./...  # Update all dependencies
go get -u=patch ./...  # Update patch versions only
```

---

## 19. SBOM Creation

### 19.1 What is an SBOM?

A Software Bill of Materials documents all dependencies and transitive modules. Critical for security audits, license tracking, and supply chain transparency.

### 19.2 CycloneDX and Go

**Using `cyclonedx-gomod`** (CycloneDX's official CLI for Go modules; the
`cyclonedx-go` repository is the Go *library* for reading/writing CycloneDX
documents, not a CLI tool):

```bash
go install github.com/CycloneDX/cyclonedx-gomod/cmd/cyclonedx-gomod@latest
cyclonedx-gomod mod -json -output sbom.json
cyclonedx-gomod mod -output sbom.xml
```

### 19.3 go.mod and go.sum Analysis

- `go.mod`: Declares direct dependencies
- `go.sum`: Cryptographic hashes for reproducible builds (include in VCS)

Generate dependency tree:
```bash
go mod graph  # Full dependency tree
go mod why <module>  # Why is this module required?
```

### 19.4 Vulnerability Scanning

**Using `govulncheck`** (official Go vulnerability scanner):
```bash
go install golang.org/x/vuln/cmd/govulncheck@latest
govulncheck ./...
govulncheck -json ./... > vuln-report.json
```

### 19.5 License Compliance

**Using `go-licenses`** (Google) — `mitchellh/golicense` is archived and no
longer maintained:

```bash
go install github.com/google/go-licenses@latest
go-licenses report ./...
go-licenses check ./... --disallowed_types=forbidden,restricted
```

### 19.6 Integration into CI/CD

- Run `govulncheck` on every commit (GitHub Actions, GitLab CI)
- Generate SBOM with `cyclonedx-gomod` on release
- Check license compliance as gate for deployment
- Store SBOM/audit reports as release artifacts

---

## 20. References

### Official Documentation

| Resource | URL |
|---|---|
| *Effective Go* | https://go.dev/doc/effective_go |
| Go Code Review Comments | https://go.dev/wiki/CodeReviewComments |
| Go Specification | https://go.dev/ref/spec |
| Go Proverbs | https://go-proverbs.github.io |
| Go Blog | https://go.dev/blog |
| Standard Library | https://pkg.go.dev/std |

### Books

| Book | Authors | Key Takeaways for This Guide |
|---|---|---|
| *The Go Programming Language* | Donovan & Kernighan (2015) | Idiomatic Go from fundamentals to concurrency; the definitive textbook. |
| *Design Patterns: Elements of Reusable Object-Oriented Software* | Gamma, Helm, Johnson, Vlissides (1994) | Favour composition over inheritance; program to interfaces — Go's design embodies these. |
| *Clean Code: A Handbook of Agile Software Craftsmanship* | Robert C. Martin (2008) | Small functions, meaningful names, SRP — applies regardless of language paradigm. |
| *Concurrency in Go* | Katherine Cox-Buday (2017) | Goroutine patterns, pipeline design, error propagation in concurrent code. |
| *100 Go Mistakes and How to Avoid Them* | Teiva Harsanyi (2022) | Practical pitfalls in error handling, concurrency, testing, and performance. |
| *The Pragmatic Programmer* | Hunt & Thomas (1999, 2019) | DRY, orthogonality, tracer bullets — universal engineering wisdom. |
