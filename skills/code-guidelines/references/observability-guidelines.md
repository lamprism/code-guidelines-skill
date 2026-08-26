# Observability And Runtime Guidelines

These requirements apply when a process, service, worker, scheduled component, command, or deployed application exposes runtime behavior that must be operated, diagnosed, or shut down safely. They cover logs, metrics, traces, health checks, lifecycle, runtime configuration, feature flags, and alerts. They use the normative model defined by `../SKILL.md` and supplement `engineering-guidelines.md`.

The goal is to make important behavior diagnosable and operable without leaking sensitive data, creating excessive cost, or turning diagnostics into an accidental business contract.

## 1. Observability Design And Ownership

**TRIGGER:** A change adds or modifies logging, metrics, tracing, health checks, runtime configuration, feature flags, alerts, startup, shutdown, or another operational behavior.

- The operator question, user impact, failure mode, signal owner, and expected action MUST be established before a signal is added.
- The existing logging, metrics, tracing, health, configuration, and alerting stack SHOULD be reused.
- Signal names, fields, units, severity, cardinality, retention, sampling, and cost SHOULD be consistent with the established project or platform conventions.
- Observability data MUST NOT become a public business contract accidentally.
- Instrumentation MUST NOT be added solely for hypothetical debugging or performance questions when no present-day operational need exists.
- Every signal MUST have a meaningful owner, or its ownership MUST remain explicitly unverified.

Observability complements tests and runtime verification; it MUST NOT be used to hide an incorrect contract, missing error handling, or an unbounded resource.

## 2. Structured Logging

- Application logs MUST use the established logging framework and structured fields when the platform supports them.
- A useful event SHOULD identify the component, operation, outcome, duration when relevant, and safe domain or request identifiers.
- Trace, request, or correlation identifiers SHOULD be preserved across an operation when the architecture provides them.
- Log levels MUST reflect operational meaning: errors that require intervention, warnings that are abnormal but handled, informational lifecycle or business events, and debug details for diagnosis.
- An error SHOULD be logged by the layer that owns the failure decision. The same exception MUST NOT produce repeated full stack traces at every layer.
- Exception causes MUST be preserved when a failure is translated or rethrown.
- High-frequency non-critical paths SHOULD avoid noisy logs that obscure actionable events.
- Log messages SHOULD be stable enough for humans and supported parsing, but consumers MUST NOT depend on incidental message text unless it is an explicit contract.

**TRIGGER:** External input, user-controlled text, identifiers, headers, or third-party data is written to logs.

- The value MUST be bounded, encoded through the structured logging API, and redacted or normalized when control characters or log injection could alter interpretation.
- Raw request bodies, credentials, access tokens, session identifiers, private keys, and unnecessary personal or sensitive data MUST NOT be logged.
- Allowlisting the fields needed for diagnosis SHOULD be preferred over logging an entire external object.

Security-sensitive logging MUST also follow `security-guidelines.md`. Application logging MUST NOT be replaced by ad hoc console printing or direct stack-trace printing.

## 3. Metrics

- A metric MUST have a clear name, unit, type, owner, and interpretation.
- Counters SHOULD represent occurrences or totals, histograms or summaries SHOULD represent latency or distribution, and gauges SHOULD represent current state when that state has operational meaning.
- Request rate, error outcomes, latency, saturation, and resource exhaustion SHOULD be measurable for an important externally reachable or operationally critical path when the platform supports metrics.
- Metric labels MUST come from a bounded, intentional set.
- User IDs, request IDs, raw URLs, arbitrary exception messages, timestamps, unbounded identifiers, and other high-cardinality values MUST NOT be used as metric labels.
- Units, bucket boundaries, aggregation behavior, and reset semantics MUST be understood before a metric is used for a decision or alert.
- A metric SHOULD be recorded at the boundary that owns the outcome, so retries and internal attempts are not accidentally counted as successful business operations.
- Metrics MUST NOT record secrets or unnecessary sensitive data.
- Instrumentation overhead, storage cost, scrape or export behavior, and failure behavior SHOULD be considered when adding high-volume metrics.

**TRIGGER:** A metric is used to make an alerting, capacity, reliability, or product decision.

- The decision threshold, evaluation window, missing-data behavior, and response owner MUST be defined.
- The metric's aggregation MUST match the question. An average MUST NOT be used when tail latency or error distribution determines the impact.

## 4. Tracing And Context Propagation

- Spans SHOULD represent meaningful operation boundaries, external calls, persistence work, or expensive stages rather than every internal method.
- Trace or correlation context SHOULD be propagated across supported request, worker, asynchronous, and callback boundaries.
- Context propagation MUST have an ownership and cleanup policy when worker threads or asynchronous tasks are reused.
- Inbound correlation or trace identifiers MUST NOT be treated as proof of identity or authorization. Security decisions MUST use the established security context.
- Trace attributes MUST be bounded and MUST NOT contain secrets, credentials, full sensitive payloads, or unnecessary personal data.
- Sampling MAY reduce cost, but sampling MUST NOT make required security, audit, or business failure evidence impossible to obtain.
- A missing trace MUST have a diagnosable fallback such as a request or operation identifier when the architecture requires correlation.

**TRIGGER:** An asynchronous task outlives the initiating call or crosses an execution boundary.

- The task's context, owner, lifetime, cancellation, and failure reporting MUST be explicit.
- Context MUST NOT leak between unrelated requests processed by a pooled worker.

## 5. Health And Readiness

Health endpoints and probes MUST represent the lifecycle state that their consumer needs. Startup, liveness, and readiness are different questions and MUST NOT be conflated.

- Startup checks SHOULD establish that required initialization has completed.
- Liveness checks SHOULD answer whether the process can make progress. They SHOULD NOT fail solely because a recoverable dependency is unavailable when the process can remain healthy and restart would not help.
- Readiness checks SHOULD answer whether the component can accept the work its contract promises. Required dependency checks MAY be included when their failure semantics are understood.
- Health checks MUST be bounded, cheap enough for their polling frequency, and free of unintended business state changes.
- Health responses MUST NOT expose credentials, internal topology, stack traces, sensitive configuration, or unnecessary infrastructure details.
- Health failure, timeout, unknown state, and recovery behavior MUST be explicit to the deployment or caller that consumes the probe.
- A health endpoint MUST have the authentication and network exposure appropriate to the information and control it provides.
- A health check MUST NOT be added as a substitute for an actual timeout, capacity limit, or recovery strategy.

## 6. Startup, Shutdown, And Runtime Lifecycle

**TRIGGER:** A component owns startup, background work, external resources, listeners, executors, schedulers, or process shutdown.

- Startup validation MUST fail clearly when required configuration, dependencies, or resources are unavailable and safe operation cannot be established.
- Startup and shutdown ordering MUST respect dependency ownership and resource lifetimes.
- Graceful shutdown MUST stop accepting new work, apply the intended drain or cancellation policy, release owned resources, and wait within a bounded lifecycle period when the runtime supports it.
- Shutdown timeout, forced termination, rejected work, interrupted work, and incomplete drain behavior MUST be observable and semantically explicit.
- Background work MUST have an owner, failure policy, cancellation policy, and lifecycle that does not depend on the initiating request remaining alive.
- Resources owned by a framework or container MUST NOT be closed by application code unless the lifecycle contract requires it.
- Lifecycle logs SHOULD record meaningful state transitions once and SHOULD include a safe operation or component identifier.

## 7. Runtime Configuration And Feature Flags

- Runtime configuration MUST have an identified source, type, validation rule, default or requiredness, owner, and effective scope.
- Invalid mandatory configuration MUST fail clearly rather than silently selecting an unsafe or misleading default.
- Secrets MUST use the approved secret-management path and MUST NOT appear in logs, metrics, traces, error messages, or diagnostic dumps.
- Configuration changes that can alter security, data integrity, capacity, routing, or compatibility MUST have explicit rollout and failure semantics.
- Dynamic configuration MAY be used when the update mechanism, consistency, validation, rollback, and stale-value behavior are understood.
- Feature flags SHOULD have a named owner, purpose, default behavior, rollout scope, and removal condition.
- A feature flag MUST NOT become an unowned permanent branch or a hidden compatibility mechanism.
- Configuration and feature-flag values MUST NOT be treated as trusted authorization or ownership context unless they are controlled by the trusted boundary that defines that decision.

## 8. Alerts And Operational Response

**TRIGGER:** A signal is used to notify an operator or automated response system.

- An alert MUST represent an actionable symptom, risk, or violated operational objective.
- Severity, notification route, ownership, deduplication, recovery behavior, and response expectation SHOULD be explicit.
- Thresholds and evaluation windows SHOULD be based on the expected workload, resource limit, contract, or observed baseline rather than arbitrary values.
- Alerts SHOULD link to enough context, dashboard information, or a runbook for the owner to begin diagnosis.
- A log message MUST NOT automatically become an alert unless its rate, severity, and action are defined.
- Alerting and dashboards MUST NOT depend on unbounded labels or incidental log text.
- Missing telemetry, exporter failure, and stale data MUST be distinguishable from a healthy zero when that distinction affects response.

## 9. Verification

**TRIGGER:** Operational behavior changes.

- Tests SHOULD verify safe redaction, stable required fields, bounded metric labels, context propagation, health failure and recovery behavior, configuration validation, and shutdown semantics as applicable.
- Health checks SHOULD be tested for dependency timeout, unavailable dependency, startup, readiness, liveness, and recovery cases when those cases affect deployment behavior.
- Lifecycle tests SHOULD verify that owned workers, subscriptions, sockets, files, and executors do not outlive their owner unexpectedly.
- Alert rules SHOULD be checked against representative success, failure, missing-data, and recovery inputs when the project provides an executable alert test path.
- Tests MUST assert operational semantics rather than incidental log formatting, metric serialization, or trace IDs.
- Apply `testing-guidelines.md` for test-level selection, isolation, fixtures, and failure handling.

Runtime, capacity, latency, and alert-quality claims MUST identify their evidence. A successful health check or a present log line MUST NOT be treated as proof that the system is reliable or secure.

## 10. Observability Review Signals

The following patterns are investigation signals, not automatic defects:

| Signal | Concern |
|---|---|
| Logging entire request or response objects | Secrets, personal data, or unbounded payloads may leak |
| Dynamic IDs used as metric labels | Cardinality and telemetry cost may grow without bound |
| Health check performs a state-changing or expensive operation | Probes may change behavior or exhaust resources |
| Liveness depends on every external dependency | A restart may amplify a recoverable dependency failure |
| Readiness always reports success | Traffic may be sent to a component that cannot honor its contract |
| Background worker has no lifecycle owner | Work, resources, or failures may outlive the component |
| Context stored in pooled thread state without cleanup | Identity or trace data may leak across requests |
| Alert on every error log | Noise may hide actionable incidents |
| Feature flag with no owner or removal condition | Temporary behavior may become permanent complexity |
| Configuration silently falls back on invalid input | Unsafe or misleading runtime behavior may be hidden |
| Repeated full stack traces at multiple layers | Diagnosis cost and log volume may grow without adding evidence |

**TRIGGER:** An observability review signal is found within the scope of a change.

Its actual effect on diagnosis, security, resource use, lifecycle, or operational response MUST be established before the signal is treated as a defect or refactoring justification.
