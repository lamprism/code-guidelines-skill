# Engineering Guidelines

These requirements apply to source code, tests, build logic, and configuration. They use the normative model defined by `../SKILL.md`.

## 1. Design And Change Principles

### Simplicity First

Current verified requirements MUST take precedence over hypothetical future needs.

- New abstractions, interfaces, strategies, factories, extension points, or configuration mechanisms MUST NOT be introduced solely for hypothetical future flexibility.
- An abstraction SHOULD be introduced only when it provides a concrete present-day benefit such as reducing real coupling, removing duplicated knowledge, separating an actual responsibility, or representing a real variation point.
- The presence of a `switch`, `Map`, literal, concrete library call, or similar implementation construct MUST NOT by itself justify refactoring.
- Established repository patterns SHOULD be reused when they correctly fit the requirement.
- Design complexity SHOULD match the current problem. Additional layers, interfaces, or configuration surfaces MUST have concrete justification.
- Readability SHOULD take precedence over compact or clever code.

**TRIGGER:** A change introduces complexity primarily for performance reasons.

- The optimization MUST be justified by an explicit requirement, expected scale, profiling, benchmark results, production metrics, or equivalent evidence.
- Complex caching, concurrency, batching, indexing, or algorithmic optimization MUST NOT be introduced for a purely hypothetical performance problem.

### Responsibility And Dependency Design

- Relevant business concepts, responsibility ownership, and dependency direction SHOULD be established before production behavior is added or materially changed.
- A class SHOULD have one coherent responsibility and one primary reason to change.
- A method SHOULD perform one coherent responsibility at one abstraction level.
- Persistence, business rules, protocol conversion, DTO construction, presentation logic, logging policy, and infrastructure concerns SHOULD NOT be mixed in one method without a concrete reason.
- Business rules SHOULD reside in objects or components that own the relevant domain responsibility instead of being duplicated across callers.
- Callers SHOULD request behavior from the object that owns a rule rather than extracting state and reimplementing the rule externally.
- Collaborator object graphs SHOULD NOT be traversed unnecessarily through long chains of indirect access.
- Components SHOULD depend on the narrowest capability required for their responsibility.
- High-level policy SHOULD NOT depend unnecessarily on concrete infrastructure details.
- Composition SHOULD be preferred over implementation inheritance.
- A concrete implementation MUST NOT be inherited from solely to reuse behavior.

**TRIGGER:** Type-based branching for the same business variation is duplicated or materially growing across multiple components.

- Polymorphism or a strategy boundary SHOULD be considered when it centralizes a real variation point and reduces duplicated conditional logic.
- A strategy hierarchy MUST NOT be introduced when there is no real variation point.

Fallback and degraded behavior MUST be explicit in a type, method name, result state, field, or contract. A fallback MUST NOT silently change the semantic meaning of a result.

Cross-currency values MUST NOT be silently converted at a 1:1 rate or relabeled as another currency.

### SOLID, DRY, And Extensibility

SOLID principles SHOULD guide responsibility and dependency decisions but MUST NOT be used as justification for speculative abstraction.

- **Open/Closed:** when a real variation point already exists and branching would otherwise spread across the system, adding an implementation or strategy SHOULD be preferred over duplicating or expanding equivalent branches in multiple locations.
- **Liskov Substitution:** an abstraction SHOULD NOT require repeated subtype probing, selective `isXxx()` checks, or unsafe downcasts for normal use.
- **Interface Segregation:** consumers SHOULD depend only on operations they actually require.
- **Dependency Inversion:** high-level policy SHOULD depend on an appropriate abstraction when a verified architectural boundary or substitutable role exists.

An abstraction MUST NOT be introduced solely to appear compliant with SOLID.

Duplicated code SHOULD be consolidated when it represents the same business knowledge, policy, or reason for change. Code that is only syntactically similar but semantically independent SHOULD NOT be merged merely to remove textual duplication. A small amount of local duplication MAY be preferable to an incorrect abstraction.

Extensibility MUST correspond to a real business or technical variation point. A broader public capability MUST NOT be introduced solely because one subdomain currently needs a local operation.

### Contracts, Compatibility, And Defensive Logic

**TRIGGER:** Caller and callee behavior disagree.

- The intended contract MUST be established from definitions, tests, documentation, protocol contracts, and actual usage before either side is changed.
- A callee contract MUST NOT be broadened merely to accommodate an invalid caller.
- Caller behavior MUST NOT be narrowed merely to accommodate an incorrect callee.

**TRIGGER:** A change introduces compatibility behavior such as fallback keys, dual reads, dual writes, aliases, legacy fields, old endpoints, or migration branches.

- A verified consumer, persisted representation, protocol requirement, or explicit compatibility requirement MUST justify the compatibility behavior.
- Speculative compatibility MUST NOT be added.

**TRIGGER:** A change introduces null guards, exception handling, retries, fallback values, or other defensive behavior.

- The failure mode MUST be possible under the verified contract.
- The handling behavior MUST have concrete semantics.
- Defensive logic MUST NOT be added merely to make code appear more robust.
- Invalid or failed state MUST NOT be silently converted into apparently valid business data.

Obsolete behavior SHOULD be replaced directly when replacement is the requirement. Parallel variants such as `XxxV2`, `newXxx()`, or `Impl2` MUST NOT be introduced merely to avoid replacing the old implementation.

### Public API And Dependencies

- Visibility SHOULD be the narrowest that satisfies actual consumers.
- Classes, methods, fields, modules, extension points, or configuration surfaces MUST NOT be made public for hypothetical future use.

**TRIGGER:** A change introduces or expands a public or cross-module contract.

- An actual consumer, explicit contract requirement, or verified architectural purpose MUST justify the exposed surface.
- Compatibility impact MUST be evaluated deliberately.

**TRIGGER:** A change introduces a third-party dependency.

- The standard library and existing project dependencies MUST first be checked for an adequate solution.
- The dependency SHOULD provide enough concrete value to justify its maintenance, security, licensing, size, build, and runtime costs.
- Unfamiliar dependency APIs MUST be verified from authoritative documentation, source, or established project usage before use.

## 2. Code Expression And Readability

### Naming

- Names MUST express business or role semantics rather than incidental implementation detail.
- Vague names such as `data`, `handler`, `helper`, `manager`, `service`, `flag`, `status`, or `process` SHOULD NOT be used when a more precise role can be named.
- Common and unambiguous domain or industry abbreviations such as `id` and `url` MAY be used.
- Boolean names MUST express true or false semantics such as `isValid`, `hasPermission`, or `canRetry`.
- Names such as `XxxV2`, `newXxx`, or `calculateImpl2` MUST NOT be used merely to preserve an obsolete implementation beside its replacement.

### Comments And Documentation

- Implementation comments SHOULD explain why a decision, constraint, workaround, or non-obvious business rule exists.
- Comments MUST NOT merely restate what the code already expresses.
- A comment that can be removed without losing information SHOULD be removed.
- When complex business logic remains difficult to understand after appropriate naming and decomposition, a limited explanatory comment MAY describe the critical behavior.
- Comments MUST NOT substitute for clearer naming or structure.
- Documentation for classes, fields, methods, functions, or public APIs MAY describe purpose, behavior, parameters, return values, invariants, and contracts.
- Documentation comments MUST use the language's standard documentation form.
- Unexplained ticket IDs, requirement labels, review versions, or internal reference codes MUST NOT substitute for explaining the underlying reason.
- Obsolete comments MUST be removed when the implementation changes.

### Language And Characters

- Technical source content SHOULD use American English for identifiers, comments, logs, internal error messages, and technical text.
- ASCII SHOULD be used for technical source content unless non-ASCII content is required by user-facing text, localization, protocol data, or domain semantics.
- Emoji, en dashes, and em dashes SHOULD NOT be used in technical source content.
- Ordinary dashes SHOULD be minimized when a colon, parentheses, or rewording is clearer.

### Control Flow And Parsing

- `if`, `for`, `while`, and equivalent control-flow bodies MUST use braces, including single-statement bodies.
- Braced bodies MUST use multi-line form.
- Guard clauses SHOULD be used when they materially improve readability.
- Deep nesting is an investigation signal, not a fixed numerical violation.
- Methods MUST NOT be split solely to satisfy an arbitrary line-count or nesting threshold.

Regular expressions SHOULD NOT be used when explicit parsing, a structured parser, typed API, string scanning, or a small state machine is clearer.

**TRIGGER:** A regular expression is introduced.

- The matching boundary and purpose MUST be understood.
- The expression SHOULD be used only when it is materially simpler than the available alternatives.
- Its performance characteristics MUST be safe for the expected input.

**EXCEPTION:** Regular expressions MAY be used as a bounded parser or fallback for inherently textual or dirty external input, including unreliable machine-generated text, when a structured representation is unavailable or inappropriate.

### Method Parameters

**TRIGGER:** A method has more than five parameters or repeatedly receives the same group of values.

- The method's responsibility SHOULD first be examined for excessive scope.
- A parameter object SHOULD be introduced only when the values form a coherent business concept such as `TransferRequest`, `PricingContext`, or `RecallQuery`.
- Unrelated values MUST NOT be mechanically wrapped in generic containers such as `FooParams` or `FooArgs` merely to reduce the visible parameter count.
- When the values have no meaningful relationship beyond being passed together, splitting the responsibility SHOULD be considered before introducing a container.

**EXCEPTION:** External protocol request and response objects MAY contain any field count required by the external contract.

## 3. Types, Data, And Boundaries

### Closed Values And Domain Types

- Stable closed value sets such as error codes, statuses, modes, categories, strategies, and types MUST use an enum or equivalent closed type representation.
- Internal business state MUST NOT be compared through raw string or numeric literals when the value belongs to a stable closed set.
- External string or numeric values MUST be converted to the closed type once at the system boundary.
- Internal business logic MUST operate on the typed value rather than repeatedly parsing the raw representation.
- Unknown external values MUST have an explicit handling strategy such as rejection, a documented unknown state, or logging plus an intentional fallback. They MUST NOT be silently ignored or discarded.
- Value objects SHOULD be used when they protect invariants, group semantically inseparable values, or remove ambiguity from important primitives.
- Primitives MUST NOT be wrapped mechanically when no semantic or correctness benefit exists.
- `null`, zero, empty strings, arbitrary constants, or other valid-looking values MUST NOT be used as hidden business-state sentinels.

### Typed Domain Boundaries

- Stable business data crossing module or architectural boundaries MUST use named typed objects.
- Raw JSON strings, generic maps, or untyped object containers MUST NOT carry known business semantics across business-module boundaries.
- External representations SHOULD be parsed once at the boundary, processed as typed objects internally, and serialized once at the output boundary.
- The same business payload SHOULD NOT be repeatedly parsed and serialized through intermediate layers.

**EXCEPTION:** Dynamic maps, JSON nodes, weakly typed containers, or similar structures MAY remain internal to protocol adapters, external dirty-input parsers, generic data infrastructure, LLM or plan parsers, and inherently dynamic integrations.

Once stable business data leaves such an implementation boundary, it MUST be converted to the appropriate typed representation.

Open row structures from intentionally generic data sources, such as arbitrary SQL query results, MAY remain dynamic.

### JSON Processing

- A project SHOULD use one standard JSON stack rather than mixing unrelated JSON libraries throughout business code.
- Stable JSON structures SHOULD be bound to named types instead of repeatedly navigated through dynamic fields.
- Dynamic JSON representations SHOULD remain inside boundaries and parser internals when genuinely required.
- Obsolete text-processing or dynamic JSON hacks and their related comments MUST be removed when typed parsing replaces them.

### Absence And State

- A method that may legitimately return no value MUST expose that possibility through the established language or project nullability or optional-value mechanism.
- Collection-returning APIs MUST return an empty collection rather than `null`.
- Null handling MUST NOT be added when it contradicts a verified non-null contract.
- Valid-looking domain values MUST NOT substitute for absence, failure, unsupported state, or fallback behavior.
- Internal mutable collections MUST NOT be exposed directly when callers could bypass the owning object's invariants.
- Defensive copies, immutable collections, or encapsulated operations SHOULD be used when callers must not mutate internal state.
- Code MUST NOT rely on iteration order from a collection implementation that does not guarantee it.

### Persistence And Data Boundaries

- Persistence entities, DAOs, repositories, query models, and persistence-vendor types SHOULD remain inside their owning persistence boundary.
- Cross-module code SHOULD depend on domain models, read models, commands, or role interfaces rather than another module's persistence internals.
- ORM, database, cache, or storage implementation details SHOULD NOT leak through core APIs unless they are explicitly part of the contract.

**TRIGGER:** A structural persistence change normalizes, denormalizes, flattens, splits, merges, or relocates stored data.

- The change MUST have a concrete justification based on ownership, lifecycle, integrity, query patterns, indexing, update behavior, concurrency, retention, or measured performance.
- Persistence structure MUST NOT be changed mechanically merely to match a preferred pattern.

## 4. Exceptions, Logging, Security, And Resources

### Exceptions

- Business exceptions MUST carry enough context to identify the failed operation, relevant entity identifiers, and a meaningful failure reason.
- Exceptions MUST NOT be used for normal business branching.

**TRIGGER:** An exception is caught.

- The current layer MUST have a concrete responsibility to recover, translate, enrich, compensate, or record the failure.
- The failure MUST be handled, appropriately recorded, or rethrown.
- The original cause MUST be preserved when wrapping.
- Broad catch blocks MUST NOT be added merely as defensive ceremony.

### Logging

When logging, metrics, traces, health checks, runtime configuration, startup, shutdown, or alerts are part of the change, apply `observability-guidelines.md`. That reference owns signal design, levels, fields, context propagation, redaction, cardinality, lifecycle, and alert semantics.

### Security And External Input

When a change crosses a trust boundary or handles untrusted input, credentials, files, serialization, network access, or security-sensitive behavior, apply `security-guidelines.md`. That reference owns trust boundaries, validation, injection, authentication, authorization, secrets, abuse controls, and security-specific verification.

### Resources, Concurrency, And Performance

When the execution model includes threads, asynchronous work, parallelism, scheduling, structured asynchronous scopes, or shared mutable state, also apply `concurrency-guidelines.md`. That reference owns execution, ownership, liveness, capacity, cancellation, ordering, and interleaving rules.

- Files, streams, network connections, database connections, locks, and similar owned resources MUST be released deterministically through the language or framework's ownership mechanism.
- Garbage collection MUST NOT be relied upon for deterministic resource release.
- External HTTP, RPC, database, or similar calls MUST have bounded timeout behavior appropriate to their contract.

**TRIGGER:** Automatic retry, redelivery, or repeated execution can occur and duplicate execution could produce incorrect behavior.

- The retry or redelivery mechanism MUST have defined limits and semantics.
- The failure MUST be established as retryable before automatic retry is enabled.
- A side-effecting operation MUST be idempotent or otherwise protected against duplicate execution.

- Repeated I/O, repeated parsing or serialization, N+1 access patterns, avoidable scans inside loops, unbounded accumulation, and clearly inappropriate algorithmic complexity SHOULD be avoided.
- Data structures and algorithms SHOULD match the actual workload.

## 5. Testing And Verification

When test strategy, fixtures, isolation, deterministic verification, contract testing, or flaky-test diagnosis is part of the change, apply `testing-guidelines.md`. That reference owns test scope, test design, fixtures, failure classification, and verification reporting.

- Code changes MUST be verified with relevant automated tests when the project provides a runnable test path.
- Runnable verification MUST NOT be replaced by static inspection when relevant runnable verification is available.

## 6. Code Smells And Boundary Rules

Code smells are investigation signals. A smell MUST NOT be treated as sufficient evidence that refactoring is required.

**TRIGGER:** A smell is encountered within the scope of a change.

- Its concrete impact MUST be established before behavior or structure is changed because of the smell.
- Unrelated smells MUST NOT be fixed during a focused task merely because they were discovered.

| Smell | Description |
|---|---|
| Type guessing | Inferring a type from string prefixes, substrings, naming patterns, or other indirect signals instead of using explicit type information. |
| Regex overreach | Extracting structured business data with regular expressions when a structured parser or typed representation should own the problem. |
| Repeated downcasting | Repeated `instanceof`, subtype probing, or downcasting against an abstraction instead of using its contract. |
| Dual-key fallback | Reading the same semantic value from multiple keys to support compatibility that has not been demonstrated as necessary. |
| Sentinel value | Using `null`, zero, empty strings, arbitrary constants, or otherwise valid-looking values to represent absence, failure, or hidden state. |
| Repeated switch | Maintaining the same enum or type branching logic in multiple places, causing one business rule to have multiple owners. |
| Dead code | Code with no meaningful execution path or code made obsolete by the current implementation. |
| Parallel replacement | Introducing `XxxV2`, `newXxx()`, `calculateImpl2()`, or similar parallel implementations instead of replacing obsolete behavior. |
| God class | A class containing multiple unrelated responsibilities and multiple independent reasons to change. |
| Nullable-field union | Modeling mutually exclusive states with multiple nullable fields that permit invalid combinations. |
| Shared mutable state | Mutable state accessed concurrently without clear ownership or synchronization. |
| Feature envy | Logic that primarily operates on another object's data and likely belongs with that object's responsibility. |
| Data clump | The same semantically related group of values repeatedly traveling together without a meaningful domain representation. |
| Primitive obsession | Repeated primitives for concepts whose invariants or semantics justify a domain type. |
| Long parameter list | A large parameter list caused by mixed responsibilities or an unmodeled coherent domain concept. |
| Flag argument | A boolean or mode parameter that switches a method between substantially different responsibilities. |
| Divergent change | One component changes for multiple unrelated reasons. |
| Shotgun surgery | One logical change requires unnecessary edits across many components because responsibility is scattered. |
| Middleman | A component that only forwards calls and provides no meaningful responsibility, boundary, policy, or abstraction. |
| Swallowed exception | A caught failure that is neither handled, appropriately recorded, nor rethrown. |
| Exception-driven control flow | Using exceptions to represent expected business alternatives. |
| Magic number | A business-significant numeric literal whose meaning is not expressed by a named constant, enum, or domain type. |
| Deep nesting | Control flow whose nesting materially increases cognitive complexity. |
| Exposed mutable collection | Returning an internal mutable collection so callers can bypass the owning object's invariants. |
| Circular dependency | Modules or components depending on each other in both directions. |
| Anemic domain model | Domain data with meaningful behavior whose rules are instead scattered through unrelated external services or utilities. |
| Copy-pasted near-duplicate logic | Multiple implementations representing the same behavior with only minor structural differences. |
| Test accommodation | Weakening assertions, expected behavior, or test isolation merely to make an incorrect implementation pass. |
| Speculative abstraction | Introducing an abstraction without a current variation point, boundary, or concrete benefit. |
| Speculative compatibility | Adding compatibility behavior without evidence of an actual compatibility requirement. |
| Speculative defense | Adding guards, catches, retries, or fallbacks for states the verified contract does not permit or without defined handling semantics. |
| Public API leakage | Exposing implementation details or widening visibility without an actual consumer. |

### Boundary Exceptions

The following **EXCEPTION** applies only to data-modeling and representation rules.

Protocol adapters, external-input parsers, generic data-access infrastructure, LLM or plan parsers, and inherently dynamic integrations MAY internally use maps, dynamic JSON nodes, weakly typed parameter containers, regular expressions, or external-protocol DTO shapes when those representations fit the boundary.

Once stable business data leaves that boundary, it MUST be converted to the appropriate typed representation.

This exception MUST NOT relax correctness, testing, exception handling, logging, security, input validation, resource management, or concurrency requirements.

A dynamic structure MUST NOT be refactored merely because it exists inside a boundary where this exception applies. The applicable rule violation and concrete impact MUST first be established.
