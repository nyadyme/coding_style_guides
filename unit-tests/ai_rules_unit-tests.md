# AI Rules: Unit Tests

Rules for coding agents when creating or modifying code and tests in this
repository. Compact, binding version of the unit testing guideline
(full version: `docs/Unit-Test-Richtlinie.md` – adjust path as needed).
If there is a conflict, the full guideline prevails.

## Principles

- Adapt to the repository's ecosystem: detect the test framework, naming
  convention, and test layout from the existing tests and follow them. Do not
  introduce a new framework without an explicit request.
- Every new or substantially changed unit (function/method/class/module) gets
  unit tests: happy path **and** non-happy path. Happy-path-only is incomplete.
- Every test is isolated, fast (milliseconds), deterministic,
  order-independent, and asserts exactly one behavior. If randomness is
  needed, seed it. *Sole exception:* fuzz runs (see Security) intentionally
  use fresh seeds; findings are pinned as deterministic regression tests.
- Structure: Arrange – Act – Assert. No logic in tests (no `if`/`for`/`switch`);
  expected values are written as constants – never computed with the same
  logic as the production code.
- The test name states the condition and the expected result; use the
  convention established in the repo (flat name, describe/it, or sentence style).

## Boundaries: where to mock

- Mock **only at the interface boundary**: database, network/HTTP, file
  system, message bus, time, randomness, environment access.
- Inject time and randomness through abstractions (Clock or similar); never
  use real time or unseeded randomness in a test (fuzz-run input generation
  excepted, see Security).
- **Never mock inside the domain**: business logic and domain-internal
  collaborators are tested for real, in memory.
- Assert state or results. Verify interactions (mock verify) only when the
  interaction itself is the contract (e.g., "sends exactly one notification").
- Test through the public interface; do not force access to private internals.

## Required test cases per unit (where applicable)

Happy path:

- typical valid call returns the correct result
- representative valid input variants (parameterized), correct side effects

Non-happy path (main focus):

- "no value": `null`/`nil`/`undefined` or empty `Option`/`Maybe` – where
  representable in the language's type system
- empty values (string/list/map, whitespace-only), wrong type/format,
  out-of-range values, type-specific edge cases (negative, `0`, `NaN`, `±Infinity`)
- boundaries: exact min/max and ±1 on each side; overflow; collections
  empty/1/many; string length 0/1/max/over max
- error signal (= the ecosystem's mechanism: exception, `Result`/`Either`/`error`,
  error code): assert the expected type or error kind **and** the message/code;
  no silent swallowing; no inconsistent state after partial execution
- dependencies (via test double at the boundary): raises an error, returns
  no/empty/unexpected result, timeout; retry/fallback behavior is correct
- state/ordering: call in an invalid state, idempotency on repeated calls,
  violated call order
- data format: malformed structure (JSON/XML/…), encoding/special characters,
  truncated/oversized/manipulated payloads
- test concurrency only if it can be controlled deterministically; otherwise
  it belongs to a higher test level
- for rejected operations, additionally verify: state is unchanged

## Security [mandatory]

- Test malicious inputs: injection patterns (SQL/command/script), path
  traversal, oversized inputs – the unit must reject or neutralize them,
  never pass them through.
- Authorization: missing/insufficient permission → rejected; access to foreign
  resources (other user/tenant ID) → denied. If enforcement lives outside the
  unit (e.g., framework middleware), still unit-test the permission rule
  itself ("who may do what").
- Error messages must not leak internals (stack traces, paths, secrets).
- **Fuzzing / property-based tests:** generally recommended for any unit with
  a non-trivial input space (parsers, decoders, validators, computations);
  **mandatory** for units that accept or parse untrusted input at a trust
  boundary. Assert invariants (no crash, only expected error signals,
  round-trip, bounded resources). **Do not fix the seed of fuzz runs** – each
  run must use a fresh seed to explore new inputs; a fixed seed would reduce
  fuzzing to a static test set. The harness must report the seed/failing
  input on failure; pin every finding as a separate deterministic regression
  test with the concrete failing input. A failing fuzz run is a finding, not
  a flaky test. Keep the iteration count (ensemble size) configurable via
  test profile or environment variable – never hardcoded: a reduced count is
  fine for local runs to keep feedback fast, but the release gate must run
  every fuzz test with at least **20,000 iterations** before anything ships
  to production. The reduced local run does not replace the full run.

## Test data

- Synthetic data only. **Never** use production data, real personal data, or
  real secrets/keys/tokens in tests and fixtures – not even "deactivated" ones.
- Minimal and expressive; use builders/factories instead of copied
  construction blocks.

## Agent workflow

- **Bug fix:** first write a failing regression test that reproduces the bug,
  then implement the fix. The test stays in place permanently.
- **Existing test turns red after your change:** analyze the root cause. Never
  adjust assertions to match broken behavior, never delete or skip tests just
  to get green. If a test is genuinely outdated, change it with an explicit
  justification.
- **Before finishing:** run at least the tests of the modules you touched.
  Finish only with a green suite; make flaky tests deterministic instead of
  rerunning until green (a failing fuzz run is a finding, not flakiness –
  see Security).
- Do not write tests that only raise the coverage metric without asserting
  behavior.

## Never

- happy-path-only tests
- mocks inside the domain / real infrastructure in a unit test
- real time, unseeded randomness (except fuzz-run input generation), network,
  or order dependence
- shared mutable state between tests
- tests without assertions
- asserting only a generic error signal (base type `Exception`/`Error`)
- computing the expected value with the same logic as the production code

## Self-check before finishing (DoD)

Confirm for every unit you touched: boundary drawn correctly (mocked only at
the edge) · happy path covered · non-happy-path cases covered (invalid inputs,
boundaries, error signal with type **and** message, dependency failures,
state/ordering) · security cases present · fuzzing at trust boundaries (iteration count configurable; release gate ≥ 20,000) ·
test data synthetic · rejected operations leave state unchanged · all tests
deterministic and green (fuzz runs excepted: fresh seeds, findings pinned) ·
test names state condition and expected result.
