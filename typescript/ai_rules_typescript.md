# TypeScript — AI Coding Rules

Apply these rules when generating or reviewing TypeScript code.

## Formatting

- Prettier is non-negotiable.
- 2 spaces indentation.
- 80 characters line limit (or 100 by team agreement).
- One blank line between top-level declarations and between class methods.
- No blank line after opening brace or before closing brace.
- One blank line between import groups (node built-ins, third-party, internal).
- Always use semicolons. Trailing commas (`trailingComma: "all"`).
- Double quotes for strings. Template literals for interpolation.

## Naming

- `PascalCase`: classes, interfaces (no `I` prefix), type aliases, enums.
- `camelCase`: functions, variables, methods.
- `UPPER_SNAKE_CASE`: module-level constants.
- `kebab-case` or `PascalCase` for files.
- No `Enum` suffix, no Hungarian notation.
- Acronyms follow casing rules: `httpClient`, `JsonParser`.
- Booleans: `isActive`, `hasPermission`, `canEdit`.

## Types

- `strict: true` in `tsconfig.json` — non-negotiable.
- Never use `any`. Use `unknown` and narrow safely.
- Avoid type assertions (`as`). Prefer type guards and narrowing.
- `interface` for object shapes/contracts. `type` for unions, intersections, computed types.
- Discriminated unions for state representation — not optional fields or booleans.
- Branded types for nominal safety: `type UserId = number & { readonly __brand: unique symbol }`.
- `import type` for type-only imports.
- String literal unions over enums. Use regular `enum` only when runtime iteration is needed (`const enum` is inlined and cannot be iterated).
- Utility types: `Partial`, `Required`, `Readonly`, `Pick`, `Omit`, `Record`.
- `readonly` properties and `Readonly<T>` for immutable data.
- Always annotate return types for exported functions.
- Exhaustive switch via `never`: `assertNever(value: never)`.
- Pass typed objects with explicit interfaces/types between modules — never `any` or untyped plain objects.
- Define interfaces or type aliases for all data crossing module boundaries.
- At system boundaries (API responses, config files, CLI args), validate and parse into typed objects (Zod, io-ts, type guards).
- Never use `as` assertions to bypass type checking — use proper validation.

## Functions

- Small, one thing. Max ~30 lines.
- Max 2 params. Beyond that, use a config object with a defined interface.
- No boolean flag arguments.
- `function` declarations for top-level. Arrow functions for inline callbacks.
- `async`/`await` always — never raw `.then()` chains.

## Documentation (TSDoc)

- `/** */` for all exported functions, classes, interfaces, types.
- `@param name - description`, `@returns`, `@throws {ErrorType}`, `@example`.
- `/** */` on individual interface/type properties.

## Error Handling

- Typed error classes extending `Error` with a `code` property.
- `Result<T, E>` discriminated union for operations where failure is expected.
- `try`/`finally` for resource cleanup. `using` (TS 5.2+) when available.
- Structured logger (`pino`/`winston`), never `console.log` in production.

## Modules & Imports

- ES modules only. No CommonJS.
- Group: node built-ins (`node:fs`), third-party, internal.
- Named exports over default exports. `import type` for types.
- No `import *`. Barrel files (`index.ts`) for clean public APIs.
- One library per concern: one HTTP client, one validator, one date lib, one logger.

## Copy Semantics

- Objects/arrays passed by reference. Spread (`{ ...obj }`, `[...arr]`) for shallow copy.
- `structuredClone()` for deep copy. Never `JSON.parse(JSON.stringify())`.
- `const` over `let`. Never `var`.

## Async & Concurrency

- `async`/`await` is the default. Always `await` promises.
- `Promise.all` for independent concurrent work. `Promise.allSettled` for all-results.
- `AbortController` for cancellation.
- Never use sync I/O in server code. `worker_threads` for CPU-intensive work.

## Testing

- Vitest or Jest — one per project.
- `describe`/`it` with scenario and outcome. Arrange-Act-Assert.
- Dependency injection over module mocking. Type-safe mocks.
- `expectTypeOf` for type testing.

## Database

- Explicit transactions (Prisma `$transaction`, Knex `.transaction`).
- Parameterised queries only. Never string interpolation in SQL.
- Connection pools with `max`/`min`/`idleTimeoutMillis`.
- Always use parameterised queries with `$1`, `?`, or named placeholders.
- Never use template literals or string concatenation to build SQL.
- Use ORM/query builder parameterisation (Prisma, Knex, TypeORM) which handles escaping.
- Validate and constrain input before it reaches the database layer.

## Patterns

- Factory: functions over constructors. Builder: fluent class.
- Singleton: module-scoped variable. Decorator: higher-order function.
- Strategy: interface or function type. Observer: type-safe EventEmitter with generics.

## Defensive Programming

- Validate all external input at system boundaries (API payloads, URL params, form data, env vars).
- Use validation libraries (Zod, io-ts, Yup) to parse and validate at boundaries.
- Validate string lengths and numeric ranges explicitly.
- Use type guards and discriminated unions for runtime type narrowing.
- Check array bounds before indexing. Use `Map.get()` with `undefined` checks.
- Sanitize input in URLs (`encodeURIComponent`), HTML (escape functions), and shell commands.
- Never use `eval()`, `new Function()`, or `innerHTML` with user input.
- `Object.freeze()` for configuration objects.

## Project Structure

- `src/` with `models/`, `services/`, `routes/`, `middleware/`, `utils/`.
- `tests/` mirroring source structure.
- `tsconfig.json` with `strict`, `noUncheckedIndexedAccess`, `verbatimModuleSyntax`.

## Tooling

- Prettier (format), ESLint + typescript-eslint (lint), `tsc --noEmit` (type check).
- Vitest/Jest (test), pnpm/npm (package manager), npm audit (security).

## Build Tools

- npm scripts in package.json for dev, build, test tasks.
- TypeScript compiler (tsc) configured via tsconfig.json.
- Bundlers: Vite (recommended modern), Webpack (complex), esbuild (fast).
- tsconfig.json: strict: true, esModuleInterop: true, skipLibCheck: true.
- Code splitting, tree-shaking, minification automatic with modern bundlers.
- Docker multi-stage builds for production TypeScript apps.

## SBOM Creation

- Lock `package-lock.json` (npm) or `yarn.lock` (Yarn) in VCS.
- Use `@cyclonedx/npm` for CycloneDX SBOM generation.
- Run `npm audit` on every PR/commit; fail if high-severity vulns.
- Use `license-report` for license compliance verification.
- GitHub Dependabot for continuous dependency scanning.
- Gate deployments on audit passes and license compliance.
