# Testing And Verification Guidelines

These requirements apply to test design, test code, fixtures, test configuration, verification strategy, and diagnosis of failing or flaky tests. They use the normative model defined by `../SKILL.md` and supplement `engineering-guidelines.md`.

The goal is to obtain reliable evidence about behavior and contracts with the smallest test scope that can detect the relevant failure. Coverage numbers, test volume, and a green build MUST NOT replace meaningful verification.

## 1. Test Strategy And Scope

Testing MUST begin with the behavior, contract, invariant, or failure mode that needs evidence.

**TRIGGER:** Production behavior, a public contract, persistence behavior, security behavior, concurrency behavior, or build behavior changes.

- The observable behavior and risk MUST be identified before a test is selected.
- New business behavior MUST have corresponding verification.
- The smallest test level that can establish the behavior SHOULD be preferred.
- Pure deterministic logic SHOULD be tested at unit level.
- Component or integration tests SHOULD be used when framework wiring, serialization, transactions, persistence, security filters, network behavior, or another real boundary is part of the contract.
- Contract tests SHOULD be used when an API, schema, or shared boundary must remain compatible with an actual consumer or provider.
- End-to-end tests SHOULD be reserved for behavior that genuinely depends on the complete deployed path.
- A broader test MUST NOT be added merely because a smaller test is harder to write or because a test framework is available.
- Coverage metrics MAY identify untested areas, but MUST NOT be the sole reason to add tests or the sole acceptance criterion.

**TRIGGER:** A change crosses a boundary whose implementation behavior is material to correctness.

- The test MUST exercise the real boundary or a verified equivalent rather than mocking away the behavior under question.
- Test doubles MAY isolate unrelated external systems, nondeterministic infrastructure, or expensive dependencies.
- A mock MUST NOT replace the core logic, invariant, serialization, authorization decision, transaction behavior, or failure path being verified.

## 2. Determinism And Isolation

- Tests MUST be independent and MUST NOT rely on execution order, shared mutable state, or a previous test's data.
- Tests SHOULD control time, randomness, generated identifiers, environment variables, filesystem locations, and external responses when those values affect assertions.
- Arbitrary sleeps, polling without a bounded condition, and timing races MUST NOT be used as the primary synchronization mechanism.
- Asynchronous tests MUST have bounded timeouts and explicit completion, failure, cancellation, and cleanup behavior.
- Concurrency tests SHOULD use controllable executors, barriers, latches, coordination primitives, or equivalent test controls instead of relying on timing. Apply `concurrency-guidelines.md` for interleavings, ownership, cancellation, and liveness.
- Tests MUST clean up resources they own, including temporary files, database rows, threads, executors, server instances, and subscriptions.
- Test isolation MUST NOT depend on an implicit local machine, developer credentials, wall-clock timezone, locale, network availability, or test execution order unless that dependency is the behavior being tested.
- Tests MUST remain bounded in time, memory, input size, and retry count.

**EXCEPTION:** A real clock, random source, network, or external service MAY be used when integration with that specific dependency is the subject of the test and the environment is explicit, safe, and controlled.

## 3. Test Data And Fixtures

- Test data SHOULD be minimal and named according to the behavior it establishes.
- Fixtures MUST NOT contain production secrets, reusable credentials, unnecessary personal data, or uncontrolled production data.
- Boundary, invalid, missing, duplicate, expired, unauthorized, and failure inputs SHOULD be covered when they can change the contract or security property.
- Time-based tests SHOULD cover relevant timezone, offset, date-only, daylight-saving, expiration, and clock-skew semantics when those semantics are part of the behavior.
- Monetary and quantity tests MUST preserve currency, unit, scale, and precision semantics when applicable.
- Shared fixtures SHOULD be immutable or rebuilt per test when mutation could leak between tests.
- A fixture builder or helper SHOULD have a coherent domain responsibility. Generic builders MUST NOT hide important defaults or make test intent difficult to see.
- Test data MUST NOT use valid-looking sentinel values to represent absence, failure, unauthorized state, or an unsupported case when the production contract distinguishes those states.

## 4. Assertions And Test Doubles

- Assertions SHOULD describe externally meaningful behavior, state transitions, contract fields, failure semantics, or invariants.
- Tests SHOULD assert the smallest result that proves the behavior and SHOULD avoid incidental formatting, private state, call order, or implementation structure.
- Internal call counts, private fields, and exact collaborator sequencing SHOULD be asserted only when they are part of the verified contract or protect a material resource or side effect.
- Expected exceptions MUST assert the relevant failure type or contract and MUST NOT pass merely because any exception was thrown.
- Failure-path tests MUST verify that invalid, unauthorized, timed-out, cancelled, rejected, or otherwise failed work does not silently produce successful business state.
- Mocks and stubs MUST have explicit behavior for relevant success and failure paths. An unstubbed default MUST NOT be mistaken for a verified external contract.
- Test doubles SHOULD be placed at architectural boundaries. A test SHOULD use a real value object, domain rule, mapper, parser, or core algorithm when that behavior is in scope.
- Snapshot or golden-file tests MAY be used when the representation itself is a meaningful contract. They MUST NOT hide important semantic assertions behind an opaque large snapshot.

## 5. Change-Specific Verification

**TRIGGER:** A bug fix is implemented.

- A regression test SHOULD reproduce the original failure before the fix when a practical path exists.
- The test MUST distinguish the verified root cause from a coincidental implementation detail.
- The regression assertion MUST remain meaningful after the smallest correct fix.

**TRIGGER:** An HTTP or shared API contract changes.

- Tests SHOULD cover request validation, response shape, status semantics, error semantics, authorization, compatibility, idempotency, ordering, pagination, and resource limits as applicable.
- Consumer or provider contract tests SHOULD be used when actual consumer compatibility cannot be established by local unit tests.

**TRIGGER:** Persistence, a schema, a migration, or a transaction changes.

- Tests MUST cover relevant existing data, constraints, transaction boundaries, rollback or partial-execution behavior, and query semantics.
- Concurrency tests SHOULD exercise the actual lost-update, duplicate, or invariant race when the change claims to prevent one.
- Migration tests MUST NOT rely only on an empty database when existing data affects the migration.

**TRIGGER:** A security-sensitive behavior changes.

- Tests MUST include relevant allowed and denied cases at the boundary where the security property is enforced.
- Invalid, expired, malformed, cross-identity, replayed, and boundary inputs SHOULD be covered when applicable.
- Tests MUST verify that a security-control failure does not silently permit the protected operation.

**TRIGGER:** An external call, retry, timeout, fallback, or degradation path changes.

- Tests SHOULD cover success, timeout, retryable failure, non-retryable failure, cancellation, partial completion, duplicate execution, and fallback semantics when applicable.
- A test MUST NOT declare a side-effecting operation safe to retry without establishing idempotency or duplicate protection.

**TRIGGER:** A runtime, health check, shutdown path, configuration, metric, log, or trace changes.

- Tests SHOULD cover startup validation, health failure behavior, graceful shutdown, redaction, context propagation, and relevant metric or log semantics without asserting incidental formatting.
- Apply `observability-guidelines.md` when operational behavior is part of the change.

## 6. Failing And Flaky Tests

**TRIGGER:** An existing test fails after a production or test change.

- The failure MUST be classified as incorrect production behavior, an incorrect test, an environment or dependency failure, a flaky test, an intentionally changed contract, or explicitly unresolved.
- The relevant test output, command, environment, and reproduction evidence SHOULD be preserved while investigating.
- Assertions, expected results, mocks, skipped tests, timeouts, retries, or coverage MUST NOT be weakened merely to make the implementation pass.
- A flaky test MUST NOT be normalized by adding arbitrary sleeps, broad retries, or unconditional reruns without identifying the nondeterministic cause.
- A test MAY be changed when the intended contract or test assumption is demonstrably incorrect, and the reason MUST be explicit.
- Skipping or disabling a test MUST have a bounded, documented reason and an owner or removal condition when the project supports such tracking.

## 7. Verification Report

Verification reporting SHOULD state:

- the focused and broader commands that ran;
- the relevant environment, test profile, or dependency state;
- which behavior or contract each important test establishes;
- failures, flaky results, and how they were classified;
- what could not be run and why.

A passing test suite MUST NOT be presented as proof of complete correctness, security, race freedom, compatibility, or performance. Static inspection MUST NOT replace runnable verification when a relevant runnable path exists.

## 8. Testing Review Signals

The following patterns are investigation signals, not automatic defects:

| Signal | Concern |
|---|---|
| Test asserts only coverage or execution | The intended behavior may remain unverified |
| Large end-to-end test for local logic | Feedback may be slow and failures hard to localize |
| Core logic fully mocked | The test may prove only mock wiring |
| Shared mutable fixture or static state | Test order or leakage may affect results |
| Fixed sleep or delay used for synchronization | The test may be flaky and miss the intended interleaving |
| Uncontrolled clock, randomness, locale, or timezone | The result may vary across runs or machines |
| Broad exception assertion | The expected failure contract may be unverified |
| Snapshot replaces semantic assertions | Important behavior may be hidden in a large representation |
| Skipped or weakened assertion after a code change | Verification may have been adapted to an incorrect implementation |
| Test uses production credentials or data | Test isolation or security boundaries may be compromised |
| Unbounded polling or retry | A failure may hang or be masked |

**TRIGGER:** A testing review signal is found within the scope of a change.

Its concrete effect on behavioral evidence, isolation, determinism, security, or maintenance MUST be established before the signal is treated as a defect or refactoring justification.
