# Java, Kotlin, And JVM Guidelines

This document is the language and runtime layer for Java, Kotlin, and related JVM code. It owns language syntax, standard-library APIs, JVM interoperability, compiler and runtime behavior, and JVM-specific testing or build concerns. It uses the normative model defined by `../SKILL.md` and supplements `engineering-guidelines.md`. Generic references remain language-neutral; apply the concurrency reference as well when execution can overlap or when asynchronous work, coroutines, scheduling, or shared mutable state is involved.

## 1. Applicability And JVM Boundaries

- Java-specific rules apply to Java sources; Kotlin-specific rules apply to Kotlin sources.
- Shared rules apply to both languages and to JVM-facing APIs.
- Compiler level, runtime, framework, formatter, static-analysis, source layout, generated-code, and module-boundary rules MUST be established from project evidence.
- A repository-specific rule such as one top-level declaration per file, a required source directory, a prohibition on top-level Kotlin declarations, or an interface-plus-`Impl` naming convention MUST NOT be inferred as a universal JVM rule. Apply it only when `project-guidelines.md`, tooling, or multiple consistent current implementations establish it.
- Java and Kotlin code MUST remain compatible with the repository's supported language level and runtime.
- Apply `concurrency-guidelines.md` whenever correctness, capacity, latency, lifecycle, or failure behavior depends on overlapping execution.

## 2. Java Language And Types

### Syntax And Type Safety

- Imports, formatting, compiler options, and static-analysis configuration MUST be followed.
- Fully qualified type names SHOULD be avoided when an unambiguous normal import is possible.

**EXCEPTION:** A fully qualified type name MAY be used to disambiguate necessary types with the same simple name or when generated or constrained source requires it.

- Raw types and unchecked conversions MUST NOT be introduced when a parameterized type can express the contract.
- An unchecked cast or `@SuppressWarnings` MUST be narrow, justified by an established invariant, and kept at the smallest possible scope.
- `var` MAY be used when the initializer makes the type obvious and the declaration remains readable. It SHOULD NOT hide a meaningful domain type or an important conversion.
- Streams SHOULD be used when they make the operation clearer, not as a mandatory replacement for a straightforward loop.
- Stream pipelines SHOULD avoid externally visible side effects and MUST define behavior for ordering and null values when those affect the contract.
- `parallelStream()` and other parallel collection operations MUST satisfy the workload and execution-model requirements in `concurrency-guidelines.md`.

### Classes And Value Semantics

- `record` SHOULD be used for immutable data carriers when the supported Java level and framework compatibility allow it and identity, mutability, or lifecycle semantics do not require a regular class.
- A record MUST NOT be used merely to make a persistence entity, mutable aggregate, or framework proxy look smaller.
- `sealed` types SHOULD be considered for a genuinely closed polymorphic hierarchy when compile-time exhaustiveness improves correctness and the supported Java level allows it.
- `equals`, `hashCode`, and `toString` MUST follow the value or identity semantics of the type.
- Mutable fields MUST NOT participate in `equals` or `hashCode` when the object can be used as a key in a hash-based collection during mutation.

### Nullability And Optional Values

- One nullability annotation system SHOULD be used consistently within a project.

**TRIGGER:** JSpecify is the established nullability system.

- `@NullMarked` SHOULD be applied at package level when that matches the project structure.
- Nullable values MUST be marked according to the established contract.
- `Optional<T>` SHOULD primarily model an optional method result.
- `Optional<T>` MUST NOT be used as a field, method parameter, or collection element when it obscures the contract or conflicts with a framework or serialization boundary.
- `Optional.get()` SHOULD NOT be used without a locally established presence invariant; an explicit fallback or meaningful exception SHOULD communicate the absence behavior.

## 3. Kotlin Language And Types

Kotlin code SHOULD use Kotlin's type and control-flow features where they make the contract clearer, while keeping public and mixed-language APIs predictable.

### Null Safety And State

- `val` SHOULD be preferred; `var` MUST be used only when mutation is part of the responsibility.
- Nullable types MUST be explicit at Kotlin boundaries.
- The not-null assertion operator (`!!`) SHOULD NOT be used in business logic.

**EXCEPTION:** It MAY be used at a verified boundary or after a locally established invariant when the failure is intentionally immediate and the alternative would hide an invalid contract.

- `lateinit` SHOULD NOT be used to bypass construction or nullability requirements when constructor injection or an explicit lifecycle can express them.
- `data class` SHOULD represent values whose equality and copy semantics are appropriate.
- `data class` MUST NOT be used for entities or objects with identity, lifecycle, or mutable invariants when generated value semantics would be misleading.
- `sealed class` or `sealed interface` SHOULD be considered for a genuinely closed state or result hierarchy when exhaustive handling improves correctness.
- `object` SHOULD represent a stateless singleton or a deliberately process-scoped owner. Global mutable state MUST NOT be introduced merely for convenient access.

### Language Features And Readability

- Top-level functions, properties, extension functions, and multiple top-level declarations MAY be used when they express a coherent package-level capability and the project permits them. They MUST NOT be prohibited or required as a universal JVM convention.
- Scope functions such as `let`, `also`, `run`, `apply`, and `with` SHOULD be used only when receiver and return-value flow remains obvious. Nested scope functions SHOULD be avoided when they obscure ownership or control flow.
- Operator overloading, implicit conversions, DSLs, and deeply chained extensions SHOULD NOT hide domain rules, mutation, blocking, or Java-visible behavior.
- Existing Java MUST NOT be rewritten to Kotlin, and existing Kotlin MUST NOT be rewritten to Java, merely because of language preference.
- KDoc and comments MUST follow the repository's documentation and language conventions. Documentation MUST explain contract, invariants, or non-obvious decisions rather than restating syntax.

## 4. Java-Kotlin Interoperability

- Public APIs shared with Java MUST expose explicit, stable types and nullability semantics.
- Kotlin default arguments, `Unit`, nullable platform types, unsigned types, function types, `Result`, and Kotlin-only collection or value abstractions MUST NOT become accidental Java-facing contracts.
- `@JvmName`, `@JvmStatic`, `@JvmOverloads`, `@Throws`, and related annotations MAY be used when a verified Java caller, framework, reflection contract, or generated API requires the resulting shape. They MUST NOT be added speculatively.
- Java callers MUST NOT be forced to infer Kotlin nullability, default-argument behavior, exception behavior, or collection mutability from implementation details.
- Kotlin code consuming Java platform types MUST establish nullability at the boundary rather than spreading unchecked assumptions through business logic.
- Java and Kotlin collections crossing a public boundary MUST have intentional mutability and ownership semantics.
- Factories, adapters, and mapping code SHOULD keep language-specific syntax close to the boundary so core domain contracts remain understandable in both languages.

**TRIGGER:** Java and Kotlin participate in the same public or cross-module contract.

- The supported callers, generated signatures, nullability annotations, checked-exception behavior, and binary/source compatibility MUST be verified.
- A Kotlin-only API shape MUST have a concrete interoperability benefit or an explicit consumer requirement.

## 5. JVM Time And Persistence

General responsibility, dependency, typed-boundary, and JSON rules belong to `engineering-guidelines.md`. Apply `database-guidelines.md` and `security-guidelines.md` when persistence or untrusted data is involved. This section retains JVM-specific time and persistence mechanisms.

### Date And Time

- New date and time code MUST use `java.time`.
- New uses of `java.util.Date` or `Calendar` MUST NOT be introduced.

**EXCEPTION:** Legacy date types MAY be used at a required compatibility boundary, but SHOULD be converted to `java.time` immediately after crossing that boundary.

- `Instant` SHOULD be used for UTC timeline timestamps when appropriate to the contract.
- `LocalDate` MUST be used for date-only business concepts.
- `LocalDateTime` MUST be used only when a date and time intentionally have no zone or offset semantics.
- `ZonedDateTime`, `OffsetDateTime`, or an explicit `ZoneId` SHOULD be used when zone semantics are part of the value or operation.
- Cross-time-zone behavior MUST carry explicit time semantics and MUST NOT depend implicitly on the server default time zone.
- Project-standard date and time formatting or parsing utilities SHOULD be reused when they exist.

### Persistence

- JPA, Hibernate, Spring Data, JDBC, and other persistence-vendor types SHOULD NOT appear in core APIs unless repository architecture explicitly defines them as part of the contract.
- JPA entities SHOULD NOT be exposed directly as cross-module domain contracts or public API models.
- Hidden lazy-loading dependencies SHOULD NOT cross architectural boundaries.

## 6. Java Runtime Mechanisms

General exception, logging, resource-ownership, timeout, and retry rules belong to `engineering-guidelines.md` and the applicable `observability-guidelines.md`, `security-guidelines.md`, or `concurrency-guidelines.md` reference.

### Java-Specific Diagnostics

- `printStackTrace()` MUST NOT be used for application or committed debugging output.
- `System.out.println` and `System.err.println` MUST NOT be used for application or committed debugging logs.

### Java Resource Ownership

- Owned `AutoCloseable` resources MUST use try-with-resources or an equivalent deterministic ownership mechanism.

## 7. JVM Concurrency Mapping

Apply the complete `concurrency-guidelines.md` for JVM concurrency. The following rules make the language-specific failure modes explicit:

### Java

- `Executor`, `ExecutorService`, scheduler, virtual-thread executor, and framework-managed pools MUST have an identified lifecycle owner and bounded downstream resource usage.
- `CompletableFuture` chains MUST make the executor, timeout, failure, and cancellation behavior explicit when the default execution context is not verified.
- `ThreadLocal` state MUST have an ownership and cleanup policy, especially when pooled workers can process unrelated requests.
- `InterruptedException` MUST be propagated or the interrupted status MUST be restored when the current layer cannot complete the interrupted operation.
- `synchronized`, `Lock`, atomic classes, and concurrent collections MUST be chosen according to the invariant and memory semantics they provide, not merely because a type is named concurrent.

### Kotlin

- Business coroutines MUST run in an owned, structured scope. `GlobalScope` or equivalent detached scope MUST NOT own business work.
- Dispatchers MUST be selected according to CPU-bound, blocking, UI, event-loop, and limited-resource behavior. Blocking calls MUST NOT run on a dispatcher that cannot accommodate them.
- `runBlocking` SHOULD be limited to synchronous bridges, tests, and explicit lifecycle boundaries; it SHOULD NOT wrap ordinary request or coroutine work.
- Coroutine cancellation MUST remain observable and cooperative. `CancellationException` or equivalent cancellation signals MUST NOT be swallowed as ordinary failures.
- `Mutex`, atomic state, thread-safe collections, immutable state, channels, `StateFlow`, and `SharedFlow` MUST be selected according to ownership, atomicity, buffering, replay, and lifecycle requirements.
- Flow buffering, conflation, collection lifetime, and backpressure MUST be explicit when they affect correctness or resource use.

## 8. Testing And Build

Apply `testing-guidelines.md` for test-level selection, fixtures, isolation, and failure handling. Apply `observability-guidelines.md` when JVM runtime signals or lifecycle behavior are part of the contract.

- The repository's established Java and Kotlin test frameworks, compiler level, formatter, static-analysis, dependency, annotation-processing, generated-source, and build conventions MUST be followed.
- JVM-specific asynchronous, cancellation, timeout, and resource-lifecycle behavior MUST be tested when it is part of the contract. Apply the deterministic testing guidance in `concurrency-guidelines.md`.
