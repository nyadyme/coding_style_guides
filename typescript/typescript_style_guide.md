# TypeScript Coding Style Guidelines

A comprehensive guide rooted in the TypeScript Handbook, the TypeScript
Do's and Don'ts, the Google TypeScript Style Guide, and established software
engineering literature — notably *Design Patterns* (Gamma et al., 1994) and
*Clean Code* (Martin, 2008).

---

## Table of Contents

1. [Philosophy](#1-philosophy)
2. [Code Layout & Formatting](#2-code-layout--formatting)
3. [Naming Conventions](#3-naming-conventions)
4. [Functions](#4-functions)
5. [Types, Interfaces & Composition](#5-types-interfaces--composition)
6. [Documentation & Comments](#6-documentation--comments)
7. [Error Handling](#7-error-handling)
8. [Modules & Imports](#8-modules--imports)
9. [Design Patterns in TypeScript](#9-design-patterns-in-typescript)
10. [Testing](#10-testing)
11. [Database Access & ACID](#11-database-access--acid)
12. [Async & Concurrency](#12-async--concurrency)
13. [Performance & Idiomatic TypeScript](#13-performance--idiomatic-typescript)
14. [Defensive Programming & Input Validation](#14-defensive-programming--input-validation)
15. [Project Structure](#15-project-structure)
16. [Tooling](#16-tooling)
17. [Build Tools](#17-build-tools)
18. [SBOM Creation](#18-sbom-creation)
19. [References](#19-references)

---

## 1. Philosophy

### 1.1 TypeScript's Purpose

TypeScript adds a **static type system** on top of JavaScript. Its goal is to
catch errors at compile time, improve editor tooling, and make large codebases
maintainable — while remaining a **strict superset** of JavaScript.

### 1.2 Guiding Principles

| Principle | Source | Summary |
|---|---|---|
| **Leverage the type system** | TypeScript Handbook | Encode invariants in types so the compiler catches violations. |
| **Strict mode, always** | TypeScript best practices | Enable `strict: true` in `tsconfig.json` — no exceptions. |
| **Explicit over implicit** | *Clean Code*, TS Do's and Don'ts | Type annotations, explicit returns, no `any`. |
| **Composition over inheritance** | *Design Patterns* Ch. 1 | Prefer interfaces, utility types, and dependency injection over class hierarchies. |
| **Single Responsibility** | *Clean Code* Ch. 10, SOLID | Every module, type, and function should have one reason to change. |
| **YAGNI** | XP / *Clean Code* | Do not build for hypothetical future requirements. |
| **Make illegal states unrepresentable** | Community convention | Use discriminated unions and branded types to prevent invalid data at the type level. |

---

## 2. Code Layout & Formatting

### 2.1 Prettier Is Non-Negotiable

All TypeScript code **must** be formatted with **Prettier**. There are no
style debates — the formatter is the authority.

- Configure via `.prettierrc` and enforce in CI.
- Do not fight the formatter.

### 2.2 Indentation

- Use **2 spaces** per indentation level (Prettier default for TS/JS).

### 2.3 Line Length

- **80 characters** (Prettier default) or **100** by team agreement.

### 2.4 Blank Lines (Prettier, Google TS Style Guide)

Prettier handles some blank-line formatting, but the programmer controls
logical grouping:

- **One blank line** between top-level declarations (functions, classes,
  interfaces, type aliases, enums).
- **One blank line** between method definitions inside a class.
- **No blank line** after the opening brace or before the closing brace of a
  class, function, or block.
- **One blank line** between import groups (node built-ins, third-party,
  internal).
- **No blank line** between consecutive `import` statements within the same
  group.
- Use **single blank lines** within functions to separate logical sections.
  Never use two or more consecutive blank lines.
- **No blank line** between a JSDoc/TSDoc comment and the declaration it
  documents.
- **No trailing blank lines** at the end of a file (a single newline
  terminator is required).

### 2.5 Semicolons

- **Always use semicolons.** Relying on ASI (Automatic Semicolon Insertion)
  creates subtle bugs.

### 2.6 Trailing Commas

Use trailing commas in multi-line structures (Prettier's `trailingComma: "all"`)
for cleaner diffs:

```typescript
const fruits = [
  "apple",
  "banana",
  "cherry",
];
```

### 2.7 Quotes

- Use **double quotes** for strings (Prettier default). Be consistent.
- Use **template literals** for interpolation:

```typescript
const message = `Hello, ${name}!`;
```

### 2.8 File Organisation

Within a `.ts` file, order elements as follows:

1. Imports
2. Type declarations (`type`, `interface`, `enum`)
3. Constants
4. Classes / functions
5. Default export (if any)

---

## 3. Naming Conventions

| Entity | Convention | Example |
|---|---|---|
| File | `kebab-case` or `PascalCase` for components | `user-service.ts`, `UserCard.tsx` |
| Class | `PascalCase` | `HttpClient`, `UserRepository` |
| Interface | `PascalCase` (no `I` prefix) | `UserService`, `Config` |
| Type alias | `PascalCase` | `UserId`, `ApiResponse` |
| Enum | `PascalCase`, members `PascalCase` | `Color.DarkBlue` |
| Function | `camelCase` | `calculateTotal()` |
| Variable | `camelCase` | `totalCount` |
| Constant | `UPPER_SNAKE_CASE` or `camelCase` | `MAX_RETRIES`, `defaultConfig` |
| Generic parameter | Single uppercase or short `PascalCase` | `T`, `K`, `TResult` |
| Private member | No prefix (use `private` keyword or `#`) | `private count`, `#internal` |
| Boolean | Reads as assertion | `isActive`, `hasPermission`, `canEdit` |

### 3.1 Naming Guidance (*Clean Code* Ch. 2, TS Do's and Don'ts)

- **No `I` prefix on interfaces.** `UserService`, not `IUserService` — this is
  not C#.
- **No `Enum` suffix.** `Status`, not `StatusEnum`.
- **Use intention-revealing names.** `elapsedTimeInMs` beats `t`.
- **No Hungarian notation.** The type system provides that information.
- **Acronyms follow `camelCase`/`PascalCase` rules.** `httpClient`, `JsonParser`
  — not `HTTPClient`, `JSONParser`.

---

## 4. Functions

### 4.1 Size and Focus (*Clean Code* Ch. 3)

- Functions should be **small** and do **one thing**.
- Extract nested logic into well-named helpers.
- If a function spans more than ~30 lines, consider splitting it.

### 4.2 Parameters

- **Fewer parameters are better.** Beyond two, use a config object with a
  defined interface:

```typescript
interface ConnectionOptions {
  host: string;
  port: number;
  timeout?: number;
  retries?: number;
}

function connect(options: ConnectionOptions): Connection {
  // ...
}
```

- **No boolean (flag) arguments.** Split into two functions or use a
  discriminated option.

### 4.3 Return Types

- **Always annotate return types** for exported functions. The compiler infers
  them, but explicit types serve as documentation and catch unintended changes.
- Return **consistent types**. Avoid `T | undefined` as a sentinel mixed with
  `null` — pick one "absent" representation.

### 4.4 Arrow Functions vs. `function`

- Use **arrow functions** for inline callbacks and short expressions.
- Use **`function` declarations** for top-level, named functions — they are
  hoisted and show up clearly in stack traces.

```typescript
// Top-level — function declaration
function processOrder(order: Order): Receipt {
  // ...
}

// Inline — arrow function
const squares = numbers.map((n) => n ** 2);
```

### 4.5 Avoid Callbacks — Prefer Promises and `async`/`await`

Callback-based APIs create deeply nested, hard-to-read code. Wrap them in
Promises and use `async`/`await` (see [Section 12](#12-async--concurrency)).

---

## 5. Types, Interfaces & Composition

TypeScript's type system is its primary value. Use it aggressively to catch
errors at compile time.

### 5.1 `type` vs. `interface`

| Use | `interface` | `type` |
|---|---|---|
| Object shapes (DTOs, contracts) | Preferred — supports declaration merging | Works, but no merging |
| Unions, intersections, mapped types | Not possible | Required |
| Extending/implementing | `extends`, `implements` | Intersection (`&`) |
| Function types | Verbose | Cleaner: `type Fn = (x: string) => void` |

**Rule of thumb:** use `interface` for object shapes and public API contracts;
use `type` for unions, intersections, and computed types.

### 5.2 Discriminated Unions — Make Illegal States Unrepresentable

```typescript
interface Loading {
  status: "loading";
}

interface Success<T> {
  status: "success";
  data: T;
}

interface Failure {
  status: "error";
  error: Error;
}

type AsyncState<T> = Loading | Success<T> | Failure;

function render(state: AsyncState<User>): string {
  switch (state.status) {
    case "loading":
      return "Loading...";
    case "success":
      return `Hello, ${state.data.name}`;
    case "error":
      return `Error: ${state.error.message}`;
  }
}
```

- The compiler ensures exhaustive handling — no `default` needed.
- Use discriminated unions instead of optional fields, boolean flags, or
  string enums to represent state.

### 5.3 Branded (Nominal) Types

TypeScript uses structural typing. Use **branded types** to prevent accidental
mixing of semantically different values:

```typescript
type UserId = number & { readonly __brand: unique symbol };
type OrderId = number & { readonly __brand: unique symbol };

function createUserId(id: number): UserId {
  return id as UserId;
}

function getUser(id: UserId): User {
  // OrderId won't be accepted here
}
```

### 5.4 Generics

Use generics to write reusable, type-safe utilities:

```typescript
function first<T>(items: readonly T[]): T | undefined {
  return items[0];
}

interface Repository<T, Id> {
  findById(id: Id): Promise<T | null>;
  save(entity: T): Promise<void>;
}
```

- Use **constraints** to narrow generic types: `<T extends HasId>`.
- Use **default type parameters** when a sensible default exists:
  `<T = string>`.

### 5.5 Utility Types

Leverage built-in utility types instead of redefining shapes:

| Type | Purpose |
|---|---|
| `Partial<T>` | All properties optional |
| `Required<T>` | All properties required |
| `Readonly<T>` | All properties readonly |
| `Pick<T, K>` | Subset of properties |
| `Omit<T, K>` | All except named properties |
| `Record<K, V>` | Object with keys K and values V |
| `Extract<T, U>` / `Exclude<T, U>` | Filter union members |
| `NonNullable<T>` | Remove `null` and `undefined` |
| `ReturnType<F>` | Infer a function's return type |
| `Parameters<F>` | Infer a function's parameter types |

### 5.6 `any`, `unknown`, and Type Assertions

- **Never use `any`** in production code. It disables type checking entirely.
- Use **`unknown`** when the type is truly unknown — it forces safe narrowing
  before use.
- **Avoid type assertions** (`as T`). If you must assert, add a comment
  explaining why. Prefer **type guards** and **narrowing** instead.

```typescript
// Bad
const data = response.body as UserData;

// Good — type guard with runtime check
function isUserData(value: unknown): value is UserData {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    "name" in value
  );
}

if (isUserData(response.body)) {
  // response.body is safely narrowed to UserData
}
```

### 5.7 Enums

Prefer **string literal unions** over enums. Use a regular `enum` only when
you need a runtime object you can iterate over; `const enum` is inlined at
compile time and cannot be iterated.

```typescript
// Preferred — zero runtime cost, exhaustive checking
type Status = "active" | "inactive" | "suspended";

// Acceptable — when you need runtime iteration (Object.values, etc.)
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}

// Avoid — numeric enums are error-prone (implicit reverse-mapping)
enum Color {
  Red,
  Green,
  Blue,
}
```

### 5.8 SOLID in TypeScript

| Principle | TypeScript Application |
|---|---|
| **S** — Single Responsibility | One module per concern; one class per concept. |
| **O** — Open/Closed | Extend via new interface implementations, generics, and higher-order functions. |
| **L** — Liskov Substitution | Any type implementing an interface must honour its contract. |
| **I** — Interface Segregation | Keep interfaces small. Compose larger APIs from smaller ones. |
| **D** — Dependency Inversion | Accept interfaces in function parameters. Inject dependencies through constructors. |

### 5.9 Typed Data Passing

Pass **typed objects** with explicit interfaces or type aliases between
modules — never `any`, `unknown` (without narrowing), or untyped plain
objects.

- **Define interfaces or type aliases** for all data that crosses module
  boundaries.
- **At system boundaries** (API responses, config files, CLI arguments),
  validate and parse into typed objects using a validation library (Zod,
  io-ts) or type guards.
- **Use branded types** to distinguish semantically different values of the
  same primitive type (e.g., `UserId` vs. `OrderId`).
- **Never use `as` type assertions** to bypass type checking — validate
  instead.

```typescript
// Bad — untyped data flows through the application
async function fetchWeather(city: string): Promise<any> {
  const res = await fetch(`https://api.weather.example/${city}`);
  return res.json(); // any
}

const data = await fetchWeather("London");
console.log(data.temperature); // no type safety

// Good — validate and parse at the boundary
import { z } from "zod";

const WeatherReport = z.object({
  city: z.string(),
  temperature: z.number(),
  humidity: z.number(),
  conditions: z.string(),
});
type WeatherReport = z.infer<typeof WeatherReport>;

async function fetchWeather(city: string): Promise<WeatherReport> {
  const res = await fetch(`https://api.weather.example/${city}`);
  return WeatherReport.parse(await res.json());
}

const report = await fetchWeather("London");
console.log(report.temperature); // fully typed
```

---

## 6. Documentation & Comments

### 6.1 TSDoc / JSDoc

Use **TSDoc** (superset of JSDoc) for documenting public APIs:

```typescript
/**
 * Retrieve a user by their primary key.
 *
 * Looks up the user in the cache first, then falls back to the database.
 *
 * @param userId - The unique identifier for the user.
 * @param includeDeleted - If true, soft-deleted users are also returned.
 * @returns The matching user object.
 * @throws {UserNotFoundError} If no user matches the given id.
 *
 * @example
 * ```typescript
 * const user = await fetchUser(42);
 * console.log(user.name);
 * ```
 */
async function fetchUser(
  userId: number,
  includeDeleted = false,
): Promise<User> {
  // ...
}
```

### 6.2 When to Document

- **All exported functions, classes, interfaces, and types** must have TSDoc.
- Internal helpers are documented only when the name and types are not
  self-explanatory.
- Interfaces and type aliases benefit from `/** */` comments on individual
  properties:

```typescript
interface Config {
  /** TCP address to listen on (default ":8080"). */
  addr: string;
  /** Maximum duration for reading a request, in milliseconds. */
  readTimeoutMs: number;
}
```

### 6.3 Comments (*Clean Code* Ch. 4)

- Comments explain **why**, not what.
- Never commit commented-out code.
- Use `// TODO:` and `// FIXME:` sparingly.
- Do not use `@ts-ignore` without an explanatory comment and a linked issue.

---

## 7. Error Handling

### 7.1 Principles

- Use **exceptions** for exceptional conditions.
- Prefer **typed error classes** over throwing strings or plain `Error`.
- **Never silently swallow errors** — catch, handle, or propagate.

### 7.2 Custom Error Classes

```typescript
class AppError extends Error {
  constructor(
    message: string,
    public readonly code: string,
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

class NotFoundError extends AppError {
  constructor(resource: string, id: string | number) {
    super(`${resource} ${id} not found`, "NOT_FOUND");
  }
}

class ValidationError extends AppError {
  constructor(
    message: string,
    public readonly field: string,
  ) {
    super(message, "VALIDATION_ERROR");
  }
}
```

### 7.3 Result Types (Alternative to Exceptions)

For operations where failure is **expected and common**, consider a `Result`
type instead of exceptions:

```typescript
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

function parseJson(input: string): Result<unknown, SyntaxError> {
  try {
    return { ok: true, value: JSON.parse(input) };
  } catch (e) {
    if (e instanceof SyntaxError) {
      return { ok: false, error: e };
    }
    throw e; // unexpected error type — re-throw
  }
}

const result = parseJson(raw);
if (result.ok) {
  console.log(result.value);
} else {
  console.error(result.error.message);
}
```

### 7.4 Resource Cleanup

TypeScript does not have `defer` or `with`. Use `try`/`finally` or the
`using` declaration (TC39 Explicit Resource Management, TypeScript 5.2+):

```typescript
// try/finally — traditional approach
const conn = await pool.connect();
try {
  await conn.query("SELECT 1");
} finally {
  conn.release();
}

// using — modern approach (TS 5.2+, requires Symbol.dispose)
await using conn = await pool.connect();
await conn.query("SELECT 1");
// conn is disposed automatically at end of scope
```

### 7.5 Logging Over `console.log`

- Use a structured logger (`pino`, `winston`) in production, never raw
  `console.log`.
- Log at the appropriate level: `debug`, `info`, `warn`, `error`.

---

## 8. Modules & Imports

### 8.1 ES Modules Only

Use **ES module syntax** (`import`/`export`). Never use CommonJS
(`require`/`module.exports`) in new TypeScript code.

### 8.2 Import Ordering

Group imports separated by blank lines:

1. **Node built-ins** (`node:fs`, `node:path`)
2. **Third-party** (`express`, `zod`, `prisma`)
3. **Internal / project** (relative paths)

```typescript
import { readFile } from "node:fs/promises";
import path from "node:path";

import express from "express";
import { z } from "zod";

import { Config } from "./config";
import { UserService } from "./services/user-service";
```

### 8.3 Import Style

- Use **named exports** over default exports — they are easier to rename,
  refactor, and search for.
- Use **`import type`** for type-only imports (enforced by
  `verbatimModuleSyntax` in `tsconfig.json`):

```typescript
import type { User } from "./models";
import { UserService } from "./services/user-service";
```

- Never use `import *` in application code — it defeats tree-shaking and makes
  dependencies opaque.

### 8.4 One Library per Concern — No Redundant Dependencies

When multiple packages accomplish the same task, **pick one and use it
consistently**:

| Overlapping packages | Pick one |
|---|---|
| `axios` / `node-fetch` / `undici` / native `fetch` | One HTTP client |
| `zod` / `yup` / `joi` / `io-ts` | One validation library |
| `dayjs` / `date-fns` / `luxon` / `moment` | One date library |
| `winston` / `pino` / `bunyan` | One logger |
| `jest` / `vitest` / `mocha` | One test runner |
| `prisma` / `drizzle` / `typeorm` / `knex` | One ORM/query builder |
| `lodash` / native methods | Native first, lodash only when needed |

### 8.5 Barrel Exports

Use `index.ts` barrel files to provide clean public APIs for packages, but
keep them flat — never re-export everything:

```typescript
// src/services/index.ts
export { UserService } from "./user-service";
export { AuthService } from "./auth-service";
export type { ServiceConfig } from "./types";
```

---

## 9. Design Patterns in TypeScript

The GoF patterns remain relevant. TypeScript's type system, interfaces, and
first-class functions provide cleaner implementations than class-heavy
approaches.

### 9.1 Creational Patterns

#### Factory Function

Prefer factory functions over constructor classes for creating objects:

```typescript
interface Notification {
  message: string;
  send(): Promise<void>;
}

function createNotification(event: Event): Notification {
  if (event.severity === "critical") {
    return new UrgentNotification(event.message);
  }
  return new StandardNotification(event.message);
}
```

#### Builder

Use a fluent builder for complex construction:

```typescript
class QueryBuilder {
  private table: string;
  private conditions: string[] = [];
  private limitValue?: number;

  constructor(table: string) {
    this.table = table;
  }

  /** Add a WHERE condition. */
  where(condition: string): this {
    this.conditions.push(condition);
    return this;
  }

  /** Set the LIMIT clause. */
  limit(n: number): this {
    this.limitValue = n;
    return this;
  }

  /** Render the final SQL string. */
  build(): string {
    let sql = `SELECT * FROM ${this.table}`;
    if (this.conditions.length > 0) {
      sql += ` WHERE ${this.conditions.join(" AND ")}`;
    }
    if (this.limitValue !== undefined) {
      sql += ` LIMIT ${this.limitValue}`;
    }
    return sql;
  }
}
```

#### Singleton

Use module-scoped variables — modules are singletons in ES:

```typescript
// config.ts — module-level singleton
let config: Config | null = null;

/** Return the application config, loading it on first access. */
export function getConfig(): Config {
  if (!config) {
    config = loadConfig();
  }
  return config;
}
```

### 9.2 Structural Patterns

#### Decorator

Use higher-order functions or TypeScript decorators:

```typescript
/** Wrap a function with retry logic. */
function withRetry<T>(
  fn: () => Promise<T>,
  maxAttempts = 3,
): () => Promise<T> {
  return async () => {
    let lastError: unknown = new Error("withRetry: maxAttempts must be >= 1");
    for (let attempt = 0; attempt < maxAttempts; attempt++) {
      try {
        return await fn();
      } catch (e) {
        lastError = e;
        // Do not sleep after the final attempt — we are about to throw.
        if (attempt < maxAttempts - 1) {
          await sleep(2 ** attempt * 1000);
        }
      }
    }
    throw lastError;
  };
}
```

#### Adapter

Implement a target interface by wrapping an incompatible type:

```typescript
interface Printer {
  print(text: string): void;
}

class LegacyPrinterAdapter implements Printer {
  constructor(private readonly legacy: LegacyPrinter) {}

  print(text: string): void {
    this.legacy.printOld(text);
  }
}
```

### 9.3 Behavioural Patterns

#### Strategy

Use interfaces or function types:

```typescript
interface Compressor {
  compress(data: Buffer): Buffer;
}

class Archiver {
  constructor(private readonly compressor: Compressor) {}

  archive(payload: Buffer): Buffer {
    return this.compressor.compress(payload);
  }
}
```

Or simply use a function type for one-method strategies:

```typescript
type CompressFn = (data: Buffer) => Buffer;

function archive(payload: Buffer, compress: CompressFn): Buffer {
  return compress(payload);
}
```

#### Observer / Event Emitter

```typescript
type Listener<T> = (data: T) => void;

class EventEmitter<Events extends Record<string, unknown>> {
  private listeners = new Map<keyof Events, Set<Listener<unknown>>>();

  /** Register a listener for the given event. */
  on<K extends keyof Events>(event: K, listener: Listener<Events[K]>): void {
    let set = this.listeners.get(event);
    if (!set) {
      set = new Set();
      this.listeners.set(event, set);
    }
    // Safe assertion: the public `on`/`emit` overloads enforce that each
    // event key K always pairs with `Listener<Events[K]>`. The internal
    // storage is widened to `Listener<unknown>` only to keep one Map.
    set.add(listener as Listener<unknown>);
  }

  /** Emit an event, invoking all registered listeners. */
  emit<K extends keyof Events>(event: K, data: Events[K]): void {
    this.listeners.get(event)?.forEach((fn) => fn(data));
  }
}

// Type-safe events
interface AppEvents {
  userCreated: User;
  orderPlaced: Order;
}

const emitter = new EventEmitter<AppEvents>();
emitter.on("userCreated", (user) => console.log(user.name));
```

---

## 10. Testing

### 10.1 Principles

- Use **Vitest** or **Jest** — pick one per project.
- **Test behaviour, not implementation.** Tests must survive refactors.
- Follow the **testing pyramid**: unit > integration > e2e.
- Tests are first-class code — same quality standards apply.

### 10.2 Naming

Test names describe the **scenario and outcome**:

```typescript
describe("withdraw", () => {
  it("throws InsufficientFundsError when balance is too low", () => {
    // ...
  });

  it("reduces the balance by the withdrawal amount", () => {
    // ...
  });
});
```

### 10.3 Structure (Arrange-Act-Assert)

```typescript
it("applies discount to the cart total", () => {
  // Arrange
  const cart = new ShoppingCart([new Item("Book", 20.0)]);
  const discount = new PercentageDiscount(10);

  // Act
  cart.applyDiscount(discount);

  // Assert
  expect(cart.total).toBe(18.0);
});
```

### 10.4 Mocking

- Prefer **dependency injection** over module mocking.
- Only mock at **system boundaries** — HTTP clients, databases, clocks.
- Use type-safe mocks (`vi.fn<...>()` in Vitest).

### 10.5 Type Testing

Test complex types with `expectTypeOf` (Vitest) or `tsd`:

```typescript
expectTypeOf<AsyncState<User>>().toMatchTypeOf<
  Loading | Success<User> | Failure
>();
```

---

## 11. Database Access & ACID

When interacting with SQL databases, every query and transaction must respect
the **ACID** properties — **Atomicity**, **Consistency**, **Isolation**, and
**Durability**.

### 11.1 ACID at a Glance

| Property | Guarantee | Violation Example |
|---|---|---|
| **Atomicity** | A transaction either completes entirely or has no effect. | A transfer debits one account but crashes before crediting the other. |
| **Consistency** | A transaction moves the database from one valid state to another. | An insert succeeds despite violating a foreign key constraint. |
| **Isolation** | Concurrent transactions do not interfere with each other. | Two requests read the same balance, both withdraw, and the final balance is wrong. |
| **Durability** | Once committed, data survives crashes and power loss. | A commit returns successfully but the data is lost after a restart. |

### 11.2 Always Use Explicit Transactions

```typescript
// Prisma
await prisma.$transaction(async (tx) => {
  await tx.account.update({
    where: { id: fromId },
    data: { balance: { decrement: amount } },
  });
  await tx.account.update({
    where: { id: toId },
    data: { balance: { increment: amount } },
  });
});

// Knex
await knex.transaction(async (trx) => {
  await trx("accounts").where({ id: fromId }).decrement("balance", amount);
  await trx("accounts").where({ id: toId }).increment("balance", amount);
});
```

### 11.3 Choose the Correct Isolation Level

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Use When |
|---|---|---|---|---|
| READ UNCOMMITTED | Possible | Possible | Possible | Almost never |
| READ COMMITTED | No | Possible | Possible | Default for most OLTP workloads |
| REPEATABLE READ | No | No | Possible | Reports needing a stable snapshot |
| SERIALIZABLE | No | No | No | Financial transactions, inventory |

### 11.4 Use Parameterised Queries

```typescript
// Yes — parameterised (Knex)
const user = await knex("users").where({ id: userId }).first();

// Yes — parameterised (raw)
const [rows] = await pool.query("SELECT * FROM users WHERE id = ?", [userId]);

// No — string interpolation
const [rows] = await pool.query(`SELECT * FROM users WHERE id = ${userId}`);
```

### 11.5 Connection Lifecycle

- Use the ORM's / driver's **connection pool**.
- **Never create a connection per request.** Use pool settings
  (`max`, `min`, `idleTimeoutMillis`).
- Clean up with `pool.end()` on process shutdown.
- Use `using` (TS 5.2+) for connection scoping when supported.

### 11.6 SQL Injection Protection

SQL injection remains one of the most dangerous and common vulnerabilities.
**Never** build SQL queries with template literals or string concatenation.

- **Always use parameterised queries** with `$1`, `?`, or named placeholders.
- Use ORM/query builder parameterisation (Prisma, Knex, TypeORM) which
  handles escaping automatically.
- Validate and constrain input before it reaches the database layer.

```typescript
// Bad — template literal (SQL injection vector)
const [rows] = await pool.query(
  `SELECT * FROM users WHERE name = '${name}'`,
);

// Good — parameterised query
const [rows] = await pool.query(
  "SELECT * FROM users WHERE name = ?",
  [name],
);

// Good — ORM parameterisation (Knex)
const user = await knex("users").where({ name }).first();

// Good — tagged template with built-in escaping (Prisma)
const users = await prisma.$queryRaw`
  SELECT * FROM users WHERE name = ${name}
`;
```

---

## 12. Async & Concurrency

### 12.1 `async` / `await` Is the Default

All asynchronous code must use `async`/`await`. Never use raw `.then()` chains
in application code:

```typescript
// Yes
async function fetchUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) {
    throw new NotFoundError("User", id);
  }
  const data: unknown = await response.json();
  return UserSchema.parse(data); // validate at boundary
}

// No — .then() chains are harder to read and debug
function fetchUser(id: number): Promise<User> {
  return fetch(`/api/users/${id}`)
    .then((r) => {
      if (!r.ok) throw new NotFoundError("User", id);
      return r.json();
    });
}
```

### 12.2 Concurrent Operations

Use `Promise.all` for independent concurrent work. Use `Promise.allSettled`
when you need all results regardless of individual failures:

```typescript
// Concurrent — all must succeed
const [users, orders] = await Promise.all([
  fetchUsers(),
  fetchOrders(),
]);

// Concurrent — collect successes and failures
const results = await Promise.allSettled(urls.map(fetchUrl));
const successes = results.filter(
  (r): r is PromiseFulfilledResult<Response> => r.status === "fulfilled",
);
```

### 12.3 Error Handling in Async Code

- **Always `await` promises** — unhandled rejections crash Node.js processes.
- Use `try`/`catch` around `await` calls.
- Never mix `async`/`await` with `.catch()` in the same function.

### 12.4 Cancellation with `AbortController`

```typescript
async function fetchWithTimeout(
  url: string,
  timeoutMs: number,
): Promise<Response> {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeoutMs);

  try {
    return await fetch(url, { signal: controller.signal });
  } finally {
    clearTimeout(timeoutId);
  }
}
```

### 12.5 Avoid Blocking the Event Loop

- Never use synchronous I/O (`readFileSync`) in server code.
- Offload CPU-intensive work to worker threads (`worker_threads`) or a
  separate service.

---

## 13. Performance & Idiomatic TypeScript

### 13.1 Immutability

Prefer `const` over `let`. Never use `var`. Use `readonly` and `Readonly<T>`
for data that should not be mutated:

```typescript
interface Config {
  readonly host: string;
  readonly port: number;
}

const items: readonly string[] = ["a", "b", "c"];
```

### 13.2 Nullish Coalescing and Optional Chaining

```typescript
// Yes — null-safe, concise; only substitutes for null/undefined
const name = user?.profile?.name ?? "Anonymous";
const retries = config.retries ?? 3;

// No — truthy checks substitute for any falsy value (0, "", false)
const retries = config.retries || 3; // bug: 0 retries becomes 3
const label = user.label || "default"; // bug: empty string becomes "default"
```

### 13.3 Destructuring

```typescript
const { name, age } = user;
const [first, ...rest] = items;

function greet({ name, role = "user" }: { name: string; role?: string }): string {
  return `Hello, ${name} (${role})`;
}
```

### 13.4 Guard Clauses

Return early rather than nesting:

```typescript
function processOrder(order: Order): Receipt {
  if (order.isCancelled) {
    throw new OrderCancelledError(order.id);
  }
  if (order.items.length === 0) {
    return Receipt.empty();
  }
  return buildReceipt(order);
}
```

### 13.5 Copy Semantics

JavaScript (and TypeScript) objects and arrays are **passed by reference**.
Assignment creates a reference, not a copy.

```typescript
const original = { name: "Alice", tags: ["admin"] };

// Reference — same object
const ref = original;
ref.name = "Bob";
// original.name === "Bob"!

// Shallow copy — new outer object, nested arrays still shared
const shallow = { ...original };
shallow.tags.push("editor");
// original.tags includes "editor"!

// Deep copy (structuredClone, Node 17+ / modern browsers)
const deep = structuredClone(original);
deep.tags.push("viewer");
// original.tags unchanged
```

| Operation | Outer | Nested | Use when |
|---|---|---|---|
| `b = a` | Shared | Shared | You want an alias |
| `{ ...a }` / `[...a]` | New | Shared | Flat structures |
| `structuredClone(a)` | New | New | Nested mutable structures |

- Use **spread** (`{ ...obj }`, `[...arr]`) for shallow copies.
- Use **`structuredClone`** for deep copies — it handles nested objects,
  `Date`, `Map`, `Set`, and circular references.
- Avoid `JSON.parse(JSON.stringify(x))` — it drops `undefined` and
  functions, converts `Date` to ISO strings (losing the `Date` type),
  loses `Map`/`Set`/`BigInt`, and throws on circular references.

### 13.6 Exhaustive Checking

Use `never` to ensure switch statements handle all union members:

```typescript
function assertNever(value: never): never {
  throw new Error(`Unhandled value: ${value}`);
}

function describe(state: AsyncState<User>): string {
  switch (state.status) {
    case "loading":
      return "Loading...";
    case "success":
      return state.data.name;
    case "error":
      return state.error.message;
    default:
      return assertNever(state);
  }
}
```

---

## 14. Defensive Programming & Input Validation

### 14.1 Validate at Boundaries

All external input — API payloads, URL parameters, form data, environment
variables, file contents — must be validated at the system boundary before it
flows into the application.

- Use **validation libraries** (Zod, io-ts, Yup) to parse and validate at
  boundaries.
- Use **type guards** and **discriminated unions** for runtime type narrowing.
- Never trust `as` assertions for external data — validate instead.

```typescript
import { z } from "zod";

const CreateUserInput = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().int().min(0).max(150),
});

function handleCreateUser(rawBody: unknown): User {
  const input = CreateUserInput.parse(rawBody); // throws on invalid
  return userService.create(input);
}
```

### 14.2 String and Numeric Validation

When validation libraries are not in scope, validate manually:

```typescript
// String length
if (input.length > 255) {
  throw new Error("Input exceeds maximum length of 255 characters");
}

// Numeric range
if (port < 1 || port > 65535) {
  throw new RangeError(`Port must be between 1 and 65535, got ${port}`);
}
```

### 14.3 Collection Safety

- Check array bounds before indexing. Use the
  `noUncheckedIndexedAccess` compiler option.
- Use `Map.get()` and check for `undefined` before using the value.

```typescript
// With noUncheckedIndexedAccess enabled:
const first = items[0]; // type is T | undefined
if (first !== undefined) {
  process(first); // safely narrowed to T
}
```

### 14.4 Input Sanitization

- Sanitize input used in URLs with `encodeURIComponent`.
- Sanitize input used in HTML with an escape function or a templating engine
  that escapes by default.
- Never pass unsanitized user input to shell commands.

```typescript
// Bad — unsanitized URL parameter
const url = `https://api.example.com/search?q=${query}`;

// Good — encoded
const url = `https://api.example.com/search?q=${encodeURIComponent(query)}`;
```

### 14.5 Dangerous APIs

- **Never** use `eval()`, `new Function()`, or `innerHTML` with user input.
  These enable cross-site scripting (XSS) and code injection.
- Use `Object.freeze()` for configuration objects to prevent accidental
  mutation.

---

## 15. Project Structure

### 15.1 Recommended Layout

```
my-app/
    package.json
    tsconfig.json
    .prettierrc
    .eslintrc.cjs
    src/
        index.ts               # entry point
        config.ts
        models/
            user.ts
            order.ts
            index.ts           # barrel export
        services/
            user-service.ts
            auth-service.ts
            index.ts
        routes/
            user-routes.ts
        middleware/
            error-handler.ts
        utils/
            date.ts
    tests/
        services/
            user-service.test.ts
        utils/
            date.test.ts
    dist/                      # compiled output (gitignored)
```

### 15.2 `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true,
    "verbatimModuleSyntax": true,
    "outDir": "dist",
    "rootDir": "src",
    "declaration": true,
    "sourceMap": true
  },
  "include": ["src"]
}
```

- **`strict: true` is non-negotiable.**
- Enable `noUncheckedIndexedAccess` to catch unsafe array/object access.
- Enable `verbatimModuleSyntax` to enforce `import type`.

### 15.3 Dependency Management

- Use a **lockfile** (`package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`).
- Separate `dependencies` from `devDependencies`.
- Audit with `npm audit` / `pnpm audit` in CI.
- Prefer exact versions in lockfiles; use caret ranges (`^`) in
  `package.json`.

---

## 16. Tooling

### 16.1 Recommended Tool Chain

| Purpose | Tool | Notes |
|---|---|---|
| Formatter | **Prettier** | Non-negotiable — end style debates |
| Linter | **ESLint** + `typescript-eslint` | Type-aware linting |
| Type checker | **`tsc --noEmit`** | Run in CI on every PR |
| Test runner | **Vitest** or **Jest** | With coverage (`v8` provider) |
| Bundler | **esbuild**, **tsup**, or **Vite** | Fast builds |
| Package manager | **pnpm** (or npm/yarn) | Fast, strict, disk-efficient |
| Security | `npm audit` / `pnpm audit` | Vulnerability scanning |
| API documentation | **TypeDoc** | Generate docs from TSDoc comments |

### 16.2 ESLint Configuration

```javascript
// .eslintrc.cjs
module.exports = {
  parser: "@typescript-eslint/parser",
  parserOptions: { project: true },
  plugins: ["@typescript-eslint"],
  extends: [
    "eslint:recommended",
    "plugin:@typescript-eslint/strict-type-checked",
    "plugin:@typescript-eslint/stylistic-type-checked",
    "prettier",
  ],
  rules: {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/no-unused-vars": ["error", { argsIgnorePattern: "^_" }],
    "@typescript-eslint/explicit-function-return-type": [
      "error",
      { allowExpressions: true },
    ],
  },
};
```

### 16.3 CI Checks

```bash
prettier --check .                  # formatting
eslint .                            # linting
tsc --noEmit                        # type checking
vitest run --coverage               # tests with coverage
npm audit --audit-level=moderate    # vulnerability scanning
```

---

## 17. Build Tools

### 17.1 npm scripts (Task Automation)

Define build, test, dev tasks in `package.json`:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "test": "vitest",
    "lint": "eslint src/",
    "format": "prettier --write ."
  }
}
```

Run with:
```bash
npm run dev
npm run build
npm test
```

### 17.2 TypeScript Compiler (tsc)

Compile TypeScript to JavaScript:
```bash
tsc  # Uses tsconfig.json
tsc --noEmit  # Check types without emitting
tsc --watch  # Watch mode
```

Minimal tsconfig.json (see Section 15.2 for the recommended full configuration):
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "verbatimModuleSyntax": true,
    "noUncheckedIndexedAccess": true
  }
}
```

**Note:** This is a minimal starter; see §15.2 for the full recommended config
with all strict-mode flags enabled.

### 17.3 Module Bundlers

**Vite** (recommended modern):
```bash
npm create vite@latest my-app -- --template react
npm run dev  # Start dev server
npm run build  # Production build
```

**Webpack** (complex projects). Note: build-tool configuration files are
commonly CommonJS even in ESM projects; if your project is `"type": "module"`,
name the file `webpack.config.cjs`:
```javascript
// webpack.config.cjs
const path = require("path");

module.exports = {
  entry: "./src/index.ts",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
  resolve: { extensions: [".ts", ".js"] },
  module: { rules: [{ test: /\.ts$/, use: "ts-loader" }] },
  devServer: { port: 3000 },
};
```

**esbuild** (ultra-fast):
```bash
esbuild src/index.ts --bundle --outfile=dist/bundle.js
esbuild src/index.ts --bundle --outfile=dist/bundle.js --watch
```

### 17.4 Build Optimization

Code splitting, tree-shaking, minification configured in bundler config. Modern bundlers handle automatically:
```bash
npm run build  # Produces optimized bundles
```

### 17.5 Docker for TypeScript Applications

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/index.js"]
```

### 17.6 CI/CD Integration

GitHub Actions:
```yaml
name: Build & Test
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - run: npm test
```

---

## 18. SBOM Creation

### 18.1 What is an SBOM?

A Software Bill of Materials documents all npm/yarn packages and transitive dependencies. Essential for vulnerability tracking, license compliance, and supply chain transparency.

### 18.2 npm and Dependency Locking

- `package.json`: Declares direct dependencies
- `package-lock.json` (npm) / `yarn.lock` (Yarn): Locks exact versions (always in VCS)

View dependency tree:
```bash
npm list  # Recursive tree
npm list --depth=0  # Direct dependencies only
npm why <package>  # Why is this included?
```

### 18.3 SBOM Generation with CycloneDX

**Using `@cyclonedx/npm`**:
```bash
npm install -g @cyclonedx/npm
cyclonedx-npm --output-file sbom.json
cyclonedx-npm --output-format xml --output-file sbom.xml
```

Or add to `package.json` scripts:
```json
"scripts": {
  "sbom": "cyclonedx-npm --output-file sbom.json"
}
```

### 18.4 Vulnerability Scanning

**Using `npm audit`** (built-in):
```bash
npm audit  # List vulnerabilities
npm audit --json > audit-report.json
npm audit fix  # Auto-fix if possible
```

**GitHub Dependabot** (native integration):
- Scans `package-lock.json` / `yarn.lock`
- Creates PRs for updates
- Shows severity and remediation

### 18.5 License Compliance

**Using `license-report`**:
```bash
npm install -g license-report
license-report --output=markdown > licenses.md
```

For policy enforcement (whitelisting allowed licenses, failing on
disallowed ones), use `license-checker` or `license-checker-rseidelsohn`:
```bash
npx license-checker --onlyAllow "MIT;Apache-2.0;BSD-2-Clause;BSD-3-Clause;ISC"
```

### 18.6 Integration into CI/CD

- Lock `package-lock.json` in VCS (required)
- Run `npm audit` on every PR; fail if high-severity vulns
- Generate SBOM on release with `@cyclonedx/npm`
- Check licenses; gate deployment on compliance
- Use GitHub Dependabot or Snyk for continuous monitoring
- Store SBOM and audit reports with release

---

## 19. References

### Official Documentation

| Resource | URL |
|---|---|
| TypeScript Handbook | https://www.typescriptlang.org/docs/handbook/ |
| TypeScript Do's and Don'ts | https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html |
| TSDoc Specification | https://tsdoc.org |
| Google TypeScript Style Guide | https://google.github.io/styleguide/tsguide.html |
| TypeScript Release Notes | https://www.typescriptlang.org/docs/handbook/release-notes/ |
| MDN Web Docs (JavaScript) | https://developer.mozilla.org/en-US/docs/Web/JavaScript |

### Books

| Book | Authors | Key Takeaways for This Guide |
|---|---|---|
| *Effective TypeScript* | Dan Vanderkam (2019, 2024) | 83 specific ways to improve your TypeScript — types, generics, inference, narrowing. |
| *Programming TypeScript* | Boris Cherny (2019) | Deep dive into the type system: mapped types, conditional types, variance. |
| *Design Patterns: Elements of Reusable Object-Oriented Software* | Gamma, Helm, Johnson, Vlissides (1994) | Favour composition over inheritance; program to interfaces. |
| *Clean Code: A Handbook of Agile Software Craftsmanship* | Robert C. Martin (2008) | Small functions, meaningful names, SRP — applies regardless of paradigm. |
| *JavaScript: The Good Parts* | Douglas Crockford (2008) | Understanding JS's quirks that TypeScript inherits and guards against. |
| *The Pragmatic Programmer* | Hunt & Thomas (1999, 2019) | DRY, orthogonality, tracer bullets — universal engineering wisdom. |
