# Unit Test Guide

*Language- and framework-agnostic requirements for writing unit tests.
Happy path and non-happy path are both mandatory; the non-happy path is
additionally relevant to quality **and** security. Ecosystem-specific
implementation is governed by Section 1.*

---

## 1. Purpose, Scope and Binding Force

This guideline defines how unit tests must be structured so that they are
understandable, maintainable and meaningful. It applies to all components,
regardless of programming language and test framework.

A unit test verifies **a single, isolated behaviour** of a unit (function,
method, class, module). It replaces neither integration nor end-to-end tests.

**Ecosystem-neutral interpretation (tailoring).** Binding are the requirements at
the behavioural level – *what* is verified. *How* this is implemented (test
framework, naming convention, error mechanism, mocking tool, fuzzing tool) is
determined by each team for its own ecosystem and documented in the team's usual
place. If a point is not technically applicable (e.g. `null` tests in languages
without `null`), it is transferred in spirit to the equivalent concept or omitted
with a brief justification.

**New and existing code.** The guideline applies in full to new and substantially
modified code. Existing code is brought up to standard incrementally as it is
changed – at minimum for the unit being touched. Retroactive full coverage is not
required.

---

## 2. Core Principles

Every unit test satisfies the following properties:

- **Isolated** – no real external dependencies (database, network, file system,
  clock, randomness). Such dependencies are replaced by test doubles
  (terminology: Section 5).
- **Fast** – an individual test runs in the millisecond range.
- **Deterministic** – the same result on every execution. No unseeded
  randomness, no dependency on real time, time zone or execution environment. If
  randomness is required, then with a **fixed seed**.
  *Sole exception:* fuzzing runs (Section 8.1) deliberately use varying seeds;
  their findings are pinned down as deterministic regression tests.
- **Independent** – order and parallelism must not influence the result. No
  shared, mutable state between tests.
- **Focused** – one test verifies exactly one behaviour / one assertion.

---

## 3. Test Boundaries: Domain and Interface Boundaries

Before a test is written, it must be clear **where the unit under test ends and
the outside world begins.** Mocking happens at this boundary – not before it and
not beyond it.

### Domain boundary (core / business logic)
- Contains business rules, calculations and domain validation – **free of
  infrastructure**.
- Is tested **for real** and entirely in memory, without mocks for business logic.
- Collaborators internal to the domain are **tested along with it**, not mocked
  away – they are part of the unit.
- This is where the greatest test depth lies, especially for the non-happy-path
  cases from Section 8.

### Interface boundary (ports / adapters to the outside world)
- The edges of the system: database, HTTP/external services, file system,
  message bus – as well as **time, randomness and environment access**.
- **Unit tests end at this boundary.** Everything beyond it is replaced by a test
  double; the real dependency is *not* tested (no testing of the database or the
  HTTP client itself).
- The **adapter** is tested for correct translation (domain object ⇄ external
  format) and for the mapping of errors from the outside world – while the
  outside world is mocked (see Section 8.4).
- Time, randomness and environment are injected via abstractions (e.g. `Clock`,
  `RandomProvider` or the respective language equivalent) and controlled in the
  test.

### Rules of thumb
- **Mock at architectural boundaries, not within the domain.**
- Assignment: business rules, calculations, decisions → *domain* (test for real).
  I/O, serialisation, protocol, persistence → *interface* (mock).
- Test the **behaviour at the boundary (the contract)**, not the specific
  technology.
- **[Security]** The interface boundary is usually also the **trust boundary**:
  data coming from outside is not trustworthy. This is exactly where input
  validation and rejection of malicious input belong under test
  (see Sections 8.1, 8.6, 8.8).

---

## 4. Structure and Naming

**Structure** follows the AAA pattern (or Given-When-Then):

1. **Arrange** – set up test data and preconditions.
2. **Act** – invoke the unit under test exactly once.
3. **Assert** – verify the result or the behaviour.

**Naming** – the binding element is the principle: the test name must convey
**condition and expected result**, so that on failure it is clear what is broken
without looking at the code. The specific convention depends on the ecosystem and
is defined **consistently across the team**, for example:

```
Scheme A – flat name:
  <unit>_<condition>_<expectedResult>
  withdraw_amountExceedsBalance_rejectsAndLeavesBalanceUnchanged

Scheme B – nested blocks (describe/context/it):
  describe(withdraw) → context(amount exceeds balance)
    → it(rejects and leaves the balance unchanged)

Scheme C – sentence form:
  "withdraw rejects and leaves the balance unchanged
   when the amount exceeds the balance"
```

**No logic in the test.** No `if`/`for`/`switch`, no computation of the expected
value using the same formula as the production code. Expected values are written
out as constants.

---

## 5. Test Doubles and Test Data

### Test doubles – consistent terminology
Regardless of the specific tool, the following terms are used:

- **Stub** – returns prepared answers; no verification.
- **Fake** – lightweight, working substitute implementation
  (e.g. in-memory repository).
- **Mock / spy** – records interactions and allows them to be verified.

Rule: **verify state or result; verify interactions only where the interaction
itself is the contract** (e.g. "sends exactly one notification"). Excessive
interaction verification couples tests to the implementation. The choice of tool
(mocking library or hand-written doubles) is ecosystem-specific and is to be kept
consistent within the team.

### Test data
- **Minimal and expressive:** set only the fields relevant to the case; use
  distinctive values that are recognisable when a failure occurs.
- **Use builders / factories** for complex objects instead of copying
  construction blocks.
- **[Security] Synthetic data only:** no production data, no real personal data
  and no real credentials, keys or tokens in test code and fixtures – not even
  "deactivated" ones.

---

## 6. Test Coverage: Happy Path and Non-Happy Path

Both areas are mandatory and complement each other:

- **Happy path** (Section 7) demonstrates that the unit correctly fulfils its
  actual purpose for valid inputs. Without it, there is no evidence that the
  function does the right thing at all.
- **Non-happy path** (Section 8) demonstrates that the unit reacts in a
  controlled manner to invalid inputs, at boundaries and when dependencies fail.

The non-happy path has a twofold significance here:

- **Quality** – most production defects do not arise in the normal case but in
  unexpected situations: incomplete data, boundary values, failing dependencies.
- **Security** – nearly every vulnerability begins with an input the developer
  did not anticipate (injection, overflows, path traversal, missing validation,
  circumvention of authorisation checks). Non-happy-path tests pin down the
  expected defensive behaviour and prevent such a gap from silently reopening
  during a later refactoring.

The guiding question beyond the happy path is not *"does it work?"* but
*"in what ways can it fail or be abused – and does it then do the right thing?"*

---

## 7. Happy-Path Tests

For every public unit, the intended normal case is covered:

- **A typical valid call** returns the correct result.
- **Representative valid variants** of the input (not just a single value), e.g.
  via parameterised tests.
- **Correct side effects** are verified: expected state change, invocation of the
  dependency with the correct arguments, return value in the expected format.
- **Multi-step flows** in the valid case (e.g. create → read → update) lead to
  the expected final state.

The happy path is the foundation – it is covered solidly, but not inflated with
dozens of near-identical variants.

---

## 8. Non-Happy-Path Tests (Main Focus)

For every public unit, the following error and edge cases are verified, insofar
as they are applicable in business and technical terms (see tailoring,
Section 1). Cases with security relevance are marked.

### 8.1 Invalid inputs
- "no value": `null` / `nil` / `undefined` or an empty `Option`/`Maybe` – insofar
  as representable in the language's type system
- empty values: empty string, empty list, empty map, `""` vs. `" "`
  (whitespace only)
- wrong type or wrong format
- values outside the permitted value range
- special cases per type: negative numbers, `0`, `NaN`, `±Infinity`
- **[Security]** malicious inputs: injection patterns (SQL, command, script),
  path traversal sequences, overlong inputs – the unit must reject or neutralise
  them, not pass them through

**Fuzzing as a supplement.** The cases listed above cover the *known*. Fuzzing
(property-based or generative testing with automatically produced random and
degenerate inputs) additionally finds the cases nobody thought of, and is
therefore explicitly recommended – and may well be used generously where the
input space is large or the risk of defects is high.

- **General recommendation:** For every unit with a non-trivial input space
  (parsers, decoders, validators, calculations), fuzzing should be considered in
  addition to the manual cases.
- **[Security] Mandatory at trust boundaries:** For units that accept or parse
  untrusted input at an interface/trust boundary (Section 3), fuzzing is
  **mandatory**.
- **Invariants instead of fixed expected values:** What is verified is not a
  specific result, but that defined properties *always* hold – e.g. "never
  crashes / raises only expected error signals", "no uncontrolled memory or
  resource consumption", "round trip `decode(encode(x)) == x`", "rejects invalid
  data without passing it through".
- **No fixed seeds in the fuzzing run:** Unlike all other tests, fuzzing
  deliberately runs with a fresh, changing seed so that each run explores new
  input space. A fixed seed would reduce fuzzing to a static set of test cases
  and defeat its purpose.
- **Pin down findings reproducibly:** On failure, the tool must output the seed
  or the failing input. Every finding is pinned down as a separate,
  deterministic regression test using the **specific input** – this regression
  test is then subject once again to the determinism principle (Section 2).
- **A fuzzing failure is a finding, not a flaky test:** It is analysed and fixed,
  not "retried until green" (distinction from Section 9).
- **Staged run counts – small locally, large for acceptance:** The number of runs
  (ensemble size) is to be kept configurable (test profile, environment variable
  or similar), never hard-coded. Locally, a reduced number may be used so that
  the suite stays fast – long waits on unit tests harm acceptance. Before
  **production acceptance**, every fuzzing test must have run green with at least
  **20,000 runs**; the reduced local run does not replace this full run
  (execution: Section 9).
- **Tool choice is ecosystem-specific:** Whether a property-based framework or a
  coverage-guided fuzzer – what matters is the principle, not the tool (see
  tailoring, Section 1).

### 8.2 Boundary testing
- lower bound and upper bound exactly
- one step below and above each (`min-1`, `min`, `min+1`, `max-1`, `max`,
  `max+1`)
- **[Security]** overflow / very large values (integer overflow, memory)
- collections: empty, exactly one element, very many elements
- strings: length 0, length 1, maximum length, excess length

### 8.3 Error and exception handling
"Error signal" means the mechanism customary in the respective ecosystem –
exception, error return value (`Result`/`Either`/`error`), error code or status.
The requirements apply regardless of the mechanism:

- the **expected** error signal is raised (correct type or correct error kind
  **and** a meaningful message / error code)
- no *wrong* or *overly generic* error signal
- no "silent" failure: invalid situations are not swallowed without comment
- partially performed operations leave no inconsistent state behind
- **[Security]** error messages do not reveal sensitive internals (stack traces,
  paths, secrets)

### 8.4 Misbehaviour of dependencies
Using test doubles, the misbehaviour of the environment at the interface boundary
(Section 3) is simulated:
- dependency raises an error signal
- dependency returns "no value", an empty or an unexpected response
- timeout or unavailability
- correct behaviour of retry, fallback and error handling logic

### 8.5 State and ordering errors
- invocation in an invalid or uninitialised state
- duplicate or repeated invocations (idempotence)
- invocation order violated (e.g. use before initialisation, access after close)

### 8.6 Authorisation and permissions [Security]
- invocation without permission or with insufficient permission is rejected
- missing authentication does not result in the operation being performed
- access to another party's resources (different user/tenant ID) is denied
- if enforcement lies outside the unit (e.g. framework middleware), the business
  authorisation rule ("who may do what") is nevertheless tested as a unit; the
  wiring is verified by a higher test level.

### 8.7 Concurrency (if applicable)
- simultaneous access by multiple callers
- race conditions and correct synchronisation
- behaviour on abort / cancellation
- concurrency is only verified in a unit test if it can be controlled
  deterministically (controlled execution / synchronisation points); otherwise
  the case belongs at a higher test level.

### 8.8 Data and format errors
- malformed inputs (invalid JSON/XML/CSV or whichever format is used)
- encoding and special characters: Unicode, emojis, control characters,
  leading/trailing whitespace
- **[Security]** truncated, incomplete or overlong volumes of data;
  deliberately manipulated structures (e.g. nested payloads)

---

## 9. Execution, CI and Metrics

- **Automation:** All unit tests run automatically, locally and in CI, on every
  change; failing tests block integration.
- **Flaky tests:** Unstable tests are not "retried until green" or permanently
  ignored, but made deterministic or removed without delay. An unstable test is
  more harmful than none at all, because it trains people to ignore red runs. To
  be distinguished from this: a failing fuzzing run is not an unstable test but a
  finding (Section 8.1).
- **Runtime:** The suite stays fast enough for frequent local execution.
  Persistently slow unit tests are usually an indication of incorrectly drawn
  boundaries (Section 3).
- **Staged execution profiles:** Locally, the suite runs with a reduced fuzzing
  scope so that feedback stays fast. The acceptance/release pipeline runs all
  fuzzing tests with at least 20,000 runs before every production release; only a
  green result there grants approval. The reduced local run does not replace the
  full acceptance run (Section 8.1).
- **Coverage is an indicator, not a goal:** Coverage metrics serve to locate
  untested branches; branch/condition coverage is more meaningful than pure line
  coverage. Specific thresholds are set by the team (see tailoring, Section 1).
  Tests that only raise the metric without asserting a behaviour are not
  permitted.

---

## 10. Best Practices

- **One assertion statement per test.** Multiple assertions are fine if they
  describe the *same* fact.
- **Use parameterised tests** for similar cases with different inputs instead of
  copying code.
- **Test behaviour, not implementation.** Tests verify the externally visible
  result, not internal details – otherwise they break with every refactoring.
- **Mock at the boundaries, not in the domain** – see Section 3.
- **Meaningful failure messages** in assertions, so that a failure is
  understandable without a debugger.
- **Verify the side effect as well:** For a rejected operation, additionally
  verify that the state has remained *unchanged*.
- **Every fixed defect gets a regression test** that reproduces the failure case
  – especially for security vulnerabilities, so that they do not silently reopen
  during later refactorings.

---

## 11. Anti-Patterns – to Be Avoided

- Tests that cover only the happy path.
- Mocking within the domain instead of at the interface boundary; dragging real
  infrastructure into the unit test.
- Testing private internals directly (forcing access to private methods/fields)
  instead of going through the public interface.
- Conditional logic or loops in test code.
- Dependency on real time, unseeded randomness (exception: fuzzing runs,
  Section 8.1), network or execution order.
- Shared, mutable state between tests.
- Tests without an assertion ("runs through = green").
- Tests that exist only to raise the coverage metric and assert no behaviour.
- Asserting only on an overly generic error signal (base type `Exception`/`Error`,
  a mere "an error occurred"), so that a wrong error also makes the test green.
- Computing the expected value with the same logic as the production code.

---

## 12. Definition of Done – Checklist

A set of tests for a unit is considered complete when:

- [ ] domain and interface boundaries have been drawn deliberately; mocking
      happens only at the boundary – Section 3;
- [ ] the happy path is covered with representative valid inputs – Section 7;
- [ ] the correct side effects of the normal case are verified;
- [ ] invalid inputs are verified ("no value", empty, wrong type, out of range) –
      Section 8.1;
- [ ] the relevant boundary values are verified – Section 8.2;
- [ ] the expected error signal behaviour is verified (type/error kind **and**
      message/code) – Section 8.3;
- [ ] the misbehaviour of dependencies is simulated and verified – Section 8.4;
- [ ] state/ordering errors are verified where applicable – Section 8.5;
- [ ] security-relevant cases are covered (malicious inputs, authorisation,
      errors without information leak) – Sections 8.1, 8.3, 8.6, 8.8;
- [ ] fuzzing is present for units with untrusted input at a trust boundary
      (generally recommended, mandatory at trust boundaries) – Section 8.1;
- [ ] every fuzzing test has run green with at least 20,000 runs before
      production acceptance – Sections 8.1, 9;
- [ ] test data is synthetic (no production data, personal data or real secrets) –
      Section 5;
- [ ] for rejected operations, the immutability of the state is verified;
- [ ] every test is isolated, deterministic and order-independent (exception to
      determinism: fuzzing runs, Section 8.1);
- [ ] the test names describe condition and expected result.
