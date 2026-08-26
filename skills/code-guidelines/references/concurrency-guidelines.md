# Concurrency And Parallelism Guidelines

These requirements apply to concurrent or asynchronous execution, including threads, executors, thread pools, futures, promises, locks, atomic state, concurrent collections, structured asynchronous scopes, reactive pipelines, parallel processing, scheduled work, background jobs, message redelivery, and shared mutable state. They use the normative model defined by `../SKILL.md` and supplement `engineering-guidelines.md`.

The goal is to make ownership, visibility, ordering, capacity, cancellation, failure, and consistency explicit. Concurrency MUST be introduced for a verified requirement, not as a default synonym for speed.

## 1. Execution Model And Decision Gate

Concurrency design MUST establish the actual execution model before a mechanism is selected. The investigation SHOULD identify:

- the unit of work and its owner;
- whether work is concurrent, parallel, asynchronous, scheduled, or merely non-blocking;
- the state shared between executions and the invariant it must preserve;
- required ordering, visibility, consistency, and isolation;
- the scope of the guarantee: one call, thread, process, runtime instance, host, or distributed system;
- maximum concurrency, queue capacity, backpressure, timeout, and rejection behavior;
- cancellation, shutdown, failure, retry, redelivery, and duplicate-execution semantics.

The simplest execution model that satisfies the verified requirement SHOULD be used. A new worker, executor, queue, asynchronous wrapper, parallel collection operation, or reactive pipeline MUST NOT be introduced solely for hypothetical throughput or responsiveness.

**TRIGGER:** A change claims a performance or scalability benefit from concurrency.

- The expected workload and bottleneck MUST be established.
- The benefit SHOULD be supported by a requirement, measurement, benchmark, profiling result, or production evidence.
- The added scheduling, synchronization, memory, debugging, and failure costs MUST be considered.

An in-process synchronization mechanism MUST NOT be presented as protection across processes, hosts, containers, or service instances. The guarantee scope MUST match the race or invariant being protected.

## 2. Ownership, Visibility, And Atomicity

Immutable data, thread confinement, ownership transfer, and message passing SHOULD be preferred over shared mutable state because they reduce the number of interleavings that can violate an invariant.

**TRIGGER:** Mutable state is accessed by more than one concurrent execution.

- The owner of the state MUST be identified.
- The state's visibility and mutation rules MUST be explicit.
- Every invariant spanning multiple reads or writes MUST have one atomicity strategy.
- Publication MUST establish visibility before another execution can observe the state.
- A thread-safe collection MUST NOT be assumed to make a multi-operation workflow atomic.

Visibility-only mechanisms MAY establish visibility where their memory semantics fit the contract, but MUST NOT be used as a substitute for atomicity of compound operations. Atomic variables, compare-and-set, locks, transactions, or another mechanism MUST be selected according to the invariant rather than by name preference.

Check-then-act, read-modify-write, and check-then-use sequences MUST be atomic when another execution can change the relevant state between steps. Application-level pre-checks MUST NOT be the sole protection for a persisted or security-sensitive invariant.

### Locking

- Lock ownership and the protected invariant MUST be documented by the code structure or contract.
- Lock scope SHOULD be as small as correctness permits.
- Code MUST use a consistent lock-ordering policy when multiple locks can be acquired.
- External I/O, callbacks, arbitrary application code, and operations that can block indefinitely SHOULD NOT run while holding a lock.
- Conditions that can be invalidated by another execution MUST be rechecked after waiting.
- A lock timeout, interruption policy, and failure behavior SHOULD be defined when lock acquisition can block materially.
- Locks MUST NOT be used as a substitute for a transaction or distributed coordination protocol whose scope exceeds the lock.

Thread-safe types MUST preserve thread safety through their public operations. Internal mutable collections, iterators, caches, and callbacks MUST NOT bypass the synchronization strategy.

## 3. Executors, Capacity, And Lifecycle

Concurrent work SHOULD run on a managed executor, dispatcher, scheduler, or framework lifecycle rather than on ad hoc threads. The owner of the execution component MUST also own its startup, capacity, shutdown, and failure policy.

- A new executor or thread pool MUST NOT be created per request, message, or method invocation.
- Thread and task counts, queue capacity, and rejection behavior MUST be bounded when work or callers can grow.
- An unbounded queue MUST NOT be used to hide overload when it can exhaust memory or create unacceptable latency.
- CPU-bound and blocking work SHOULD use separate capacity when they would otherwise starve each other.
- External resource limits such as database connections, remote-service quotas, file descriptors, and rate limits MUST be included in the concurrency bound.
- Queueing, rejection, shedding, or backpressure behavior MUST be explicit to the caller or owner of the work.
- Worker names, metrics, and ownership SHOULD make the execution context diagnosable.

**TRIGGER:** A component starts background work or owns an executor.

- Startup and shutdown order MUST be defined.
- Shutdown MUST stop accepting new work, apply the intended cancellation or drain policy, and wait for completion within a bounded lifecycle period when the runtime supports it.
- Work that outlives its initiating request MUST have an explicit owner and durable failure or retry semantics.
- Detached fire-and-forget work MUST NOT be used when its result, failure, or resource ownership is part of the requested operation.

Specialized execution models, event-loop dispatchers, and runtime-managed workers MAY be used when the runtime and project support them. They do not remove the need to bound downstream resources, define cancellation, or verify the actual blocking and scheduling behavior.

## 4. Async Results, Cancellation, And Failure

Asynchronous APIs MUST preserve the relationship between a submitted operation and its result. Completion, failure, timeout, and cancellation MUST remain observable by the component that owns the work.

- An asynchronous result MUST have explicit success, failure, timeout, and cancellation semantics.
- Exceptions MUST NOT disappear in an ignored future, promise, callback, task, or asynchronous scope.
- Blocking on an asynchronous result MUST NOT occur on the same constrained executor or event loop that must complete that result.
- Every wait on external or potentially unbounded work MUST have an appropriate timeout or an explicit lifecycle reason for not having one.
- Timeouts MUST define whether work is cancelled, allowed to finish, retried, or reported as unknown.
- Cancellation SHOULD be cooperative and MUST propagate through child work and owned resources when the execution model supports propagation.
- Cancellation MUST NOT be reported as successful business completion.

Interruption, cancellation, and equivalent stop signals MUST remain observable by the component that owns the work. A caught stop signal MUST be propagated, restored, or handled according to the execution model; it MUST NOT be silently swallowed. Broad exception handling MUST NOT convert cancellation into ordinary success or keep cancelled work running.

Asynchronous scopes MUST have an owner and lifecycle. Global or detached scopes MUST NOT own business work without an explicit application-level lifecycle contract.

Child tasks SHOULD be structured under the lifetime of their parent operation. A parent MUST define whether one child failure cancels siblings, is isolated, or becomes a partial result. Supervisor-style behavior MUST be intentional rather than an accidental consequence of the framework.

## 5. Ordering, Idempotency, And Distributed Scope

Ordering MUST be treated as a contract only when the execution mechanism, queue, broker, database, or API explicitly provides it. Scheduling order, thread creation order, map iteration order, and completion order MUST NOT be assumed to be business order without evidence.

**TRIGGER:** Work can be retried, redelivered, duplicated, resumed, or observed more than once.

- The operation MUST be idempotent or protected by an atomic deduplication or idempotency mechanism.
- The mechanism MUST define the uniqueness scope, retention or expiration, replay result, and failure behavior.
- A successful local submission MUST NOT be treated as successful business execution unless the contract says so.
- Partial completion and recovery MUST be defined for multi-step work.

Persisted invariants MUST use the database, transaction, constraint, optimistic version, atomic update, or another mechanism with the required scope. An in-memory lock MUST NOT protect a rule that can be violated by another process or service instance.

Distributed locks, leases, leader election, and consensus mechanisms MUST NOT be introduced without establishing the failure model, ownership, expiration, fencing or stale-owner behavior, recovery, and operational requirements. A database lock MUST NOT be assumed to coordinate an external system.

## 6. Liveness And Blocking

Concurrency correctness includes making progress. The design MUST consider deadlock, livelock, starvation, priority inversion, thread-pool exhaustion, queue growth, and event-loop blockage when those failure modes are possible.

- Potentially blocking work MUST NOT run on an event loop, UI thread, or latency-critical executor unless the execution model explicitly permits it.
- Nested waits, synchronous calls from callbacks, and cross-pool dependencies SHOULD be inspected for deadlock and starvation.
- Lock acquisition and external calls SHOULD have bounded waits when indefinite blocking can exhaust a shared resource.
- Retries MUST NOT create unbounded concurrent work or amplify overload.
- Backoff and jitter SHOULD be used when repeated concurrent retries could synchronize and overload a dependency.
- Resource cleanup MUST remain reliable when work is cancelled, interrupted, rejected, or times out.

## 7. Testing And Observability

Apply `testing-guidelines.md` for test-level selection, fixtures, isolation, and failure classification, and apply `observability-guidelines.md` for telemetry, lifecycle, and operational signal semantics. This section owns concurrency-specific interleavings, cancellation, and liveness verification.

Concurrency tests MUST verify invariants and lifecycle behavior rather than relying on one favorable scheduling order.

- Tests SHOULD use controllable executors, clocks, barriers, latches, channels, or equivalent coordination primitives instead of arbitrary sleeps for synchronization.
- Tests SHOULD cover relevant interleavings, concurrent updates, cancellation, timeout, rejection, shutdown, failure propagation, and duplicate execution.
- Stress, repetition, or race-detection tests MAY supplement deterministic tests, but one passing run MUST NOT establish the absence of a race.
- Tests MUST remain bounded and independent; a flaky timing assumption MUST NOT be normalized as test behavior.
- Task failures, cancellations, queue saturation, active work, latency, retries, and rejected work SHOULD be observable when operational diagnosis requires them.
- Logs and metrics MUST preserve correlation without recording secrets or unnecessary sensitive data.

Verification SHOULD include the narrowest relevant concurrency test first, followed by broader tests and static analysis when available. Claims about race freedom, throughput, latency, or capacity MUST identify the evidence supporting them.

## 8. Concurrency Review Signals

The following patterns are investigation signals, not automatic defects:

| Signal | Concern |
|---|---|
| Ad hoc worker creation in request or message handling | Lifecycle, capacity, and failure ownership may be missing |
| Executor created inside a method | Pools may leak or multiply without bounded ownership |
| Unbounded queue or unconstrained fan-out | Overload may become memory exhaustion or latency collapse |
| Shared mutable state without an owner | Visibility or invariant protection may be absent |
| Visibility-only mechanism for a compound state transition | Visibility may be mistaken for atomicity |
| Check followed by separate update | Another execution may win the race |
| Parallel collection operation without workload evidence | Parallel overhead or shared-pool contention may outweigh benefit |
| Ignored asynchronous result or task | Failure may disappear |
| Empty interruption or cancellation catch | Shutdown and cancellation may be broken |
| Detached business asynchronous scope | Work may outlive its owner and leak failures |
| Blocking call on an event loop or constrained pool | Starvation or deadlock may occur |
| Local lock used for cross-instance correctness | The actual race may remain unprotected |
| Retry without idempotency or capacity bound | Duplicate effects or retry amplification may occur |
| Fixed sleep or delay used to coordinate a test | The test may be flaky and fail to cover the intended interleaving |

**TRIGGER:** A concurrency review signal is found within the scope of a change.

Its actual execution model, protected invariant, ownership, failure behavior, and operational impact MUST be established before the signal is treated as a defect or refactoring justification.
