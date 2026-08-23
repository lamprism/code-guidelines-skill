# Engineering Guidelines

These requirements apply to source code, tests, build logic, and configuration unless a more specific project or language rule explicitly overrides them.

## 1. Design And Change Principles

### Simplicity First

Follow SOLID, DRY, Clean Code, and related engineering principles, but prioritize current verified requirements over hypothetical future needs.

- **Do not abstract prematurely**: when only one implementation exists and no concrete second implementation is required, do not introduce an interface, strategy, factory, extension point, or configuration mechanism solely for future flexibility.
- **Abstractions must provide concrete value**: introduce an abstraction only when it reduces real coupling, removes duplication of the same knowledge, separates an actual responsibility, or represents a real variation point.
- **Do not refactor for appearances**: the presence of a `switch`, `Map`, literal, concrete library call, or other implementation construct is not itself a reason to refactor.
- **Prefer established project patterns**: when the repository already has a suitable approach, use it instead of introducing a competing mechanism based on personal preference.
- **Match design scale to actual requirements**: the number of abstractions, interfaces, layers, and configuration options must reflect the complexity of the current problem.
- **Prefer readability over cleverness**: simple does not mean fewer lines. Do not compress code at the cost of clarity.
- **Do not optimize hypothetical problems**: complex caching, concurrency, batching, indexing, or algorithmic optimization requires evidence from requirements, expected scale, profiling, benchmarks, or runtime observations.

Before introducing an abstraction, configuration option, compatibility path, fallback, or optimization, ask whether the corresponding variation, requirement, failure mode, or performance problem exists today. If not, do not introduce it.

### Responsibility And Dependency Design

- Design before coding. Identify the relevant business concepts, responsibility ownership, and dependency direction before adding production logic.
- Give each class one clear responsibility and one primary reason to change.
- A responsibility description containing multiple unrelated concerns is a signal to examine whether the class should be split. It is not a mechanical requirement to create more classes.
- Keep methods focused on one responsibility and one abstraction level.
- Do not unnecessarily mix persistence, business rules, protocol conversion, DTO construction, presentation logic, logging policy, and infrastructure concerns.
- Prefer guard clauses and focused helper methods when they materially reduce nesting or mixed responsibilities.
- Keep business rules inside objects with clear domain responsibilities instead of scattering equivalent conditions across callers.
- Follow Tell, Don't Ask: request behavior from the object that owns the rule instead of extracting its state and reimplementing the rule elsewhere.
- Follow the Law of Demeter. Avoid traversing collaborator object graphs such as `a.getB().getC().getD()`.
- Depend on the narrowest abstraction required to perform the responsibility.
- High-level modules must not depend unnecessarily on concrete infrastructure details.
- Prefer composition over inheritance.
- Do not inherit from a concrete implementation merely to reuse behavior.
- Use polymorphism when a real variation point exists and it reduces meaningful conditional complexity. Do not create a strategy hierarchy for a single hypothetical alternative.
- Make fallback and degraded behavior explicit in types, method names, result states, fields, or contracts. Do not silently change semantics.
- Do not silently convert values between currencies at a 1:1 rate or relabel native amounts as amounts in another currency.

### SOLID, DRY, And Extensibility

Apply SOLID as a design tool, not as a reason to maximize abstraction.

- **Single Responsibility**: one component should own one coherent responsibility and one primary reason to change.
- **Open/Closed**: when a real variation point already exists and branching would otherwise spread across the system, prefer adding a new implementation or strategy over repeatedly modifying duplicated type branches.
- **Liskov Substitution**: code using an abstraction must not depend on repeated subtype probing, selective `isXxx()` checks, or unsafe downcasts to make the abstraction usable.
- **Interface Segregation**: consumers should depend only on the operations they actually need.
- **Dependency Inversion**: high-level policy should depend on appropriate abstractions rather than directly constructing infrastructure implementations when a real architectural boundary exists.
- Do not introduce an abstraction solely to appear compliant with SOLID.
- Eliminate duplication when duplicated code represents the same business knowledge, policy, or reason for change.
- Do not merge code that is merely syntactically similar but semantically independent.
- A small amount of local duplication is preferable to an incorrect abstraction.
- Extensibility must correspond to a real business or technical variation point.
- Do not create a broader public capability solely because a subdomain currently needs one operation. Keep behavior within the owning domain or adapter unless a meaningful broader abstraction actually exists.

### Contracts, Compatibility, And Defensive Logic

- **Establish the contract before fixing a mismatch**: when caller and callee behavior disagree, use definitions, tests, documentation, protocol contracts, and actual usage to determine which side violates the intended contract.
- Do not broaden a callee's contract merely to accommodate an incorrect caller.
- Do not tighten a caller's behavior merely to accommodate an incorrect callee.
- **Do not add unverified compatibility behavior**: fallback keys, dual reads, dual writes, aliases, legacy field support, old endpoint handling, and migration branches require evidence that an actual consumer or persisted representation depends on them.
- **Defensive logic must address a real failure mode**: add `try-catch`, null checks, retries, fallback values, or validation only when the failure can actually occur under the verified contract and a concrete handling strategy exists.
- Do not add defensive code merely to make the implementation appear more robust.
- Do not silently replace invalid or failed state with apparently valid business data.
- When compatibility or fallback behavior is required, make its semantics explicit.
- Do not preserve obsolete implementations by creating parallel `V2`, `newXxx`, or `Impl2` variants when the task requires replacing the existing behavior.

### Public API And Dependencies

- Use the narrowest visibility that satisfies actual consumers.
- Do not make classes, methods, fields, modules, or extension points public for hypothetical future use.
- New public contracts must have a real consumer, explicit requirement, or verified architectural purpose.
- Treat public APIs and cross-module contracts as compatibility surfaces. Change them deliberately.
- Before adding a third-party dependency, confirm that the standard library and existing project dependencies do not already provide an adequate solution.
- Add a dependency only when its concrete value justifies its maintenance, security, licensing, size, build, and runtime costs.
- Verify unfamiliar dependency APIs before use. Do not infer method signatures or semantics from memory.

## 2. Code Expression And Readability

### Naming

- Use semantic, role-revealing names based on business meaning rather than implementation detail.
- Avoid vague names such as `data`, `handler`, `helper`, `manager`, `service`, `flag`, `status`, or `process` when a more precise role can be named.
- Common and unambiguous domain or industry abbreviations such as `id` and `url` are acceptable.
- Boolean variables and methods must express true or false semantics, such as `isValid`, `hasPermission`, or `canRetry`.
- Do not use names such as `XxxV2`, `newXxx`, or `calculateImpl2` to avoid replacing an obsolete implementation.

### Comments And Documentation

- Implementation comments should explain why a decision, constraint, workaround, or non-obvious business rule exists. Do not restate what the code already expresses.
- If deleting a comment would lose no information, the comment should not exist.
- When complex business logic remains difficult to understand after appropriate naming and decomposition, a limited explanatory comment may describe what the critical logic does.
- Do not use comments as a substitute for clearer naming or structure.
- Class, field, method, function, and public API documentation may describe purpose, behavior, parameters, return values, invariants, and contracts.
- Documentation comments must use the language's standard multi-line documentation form rather than ordinary single-line comments.
- Do not use unexplained ticket numbers, requirement labels, review versions, or internal reference codes as substitutes for explaining the underlying reason.
- When code changes, remove obsolete comments instead of leaving historical explanations behind.

### Language And Characters

- Use American English for identifiers, comments, logs, internal error messages, and technical text in source files.
- Use ASCII characters in technical source content unless non-ASCII content is required by user-facing text, localization, protocol data, or domain semantics.
- Do not use emoji, en dashes, or em dashes in technical source content.
- Minimize ordinary dashes in prose and comments. Prefer colons, parentheses, or rewording when clearer.

### Control Flow And Parsing

- `if`, `for`, `while`, and equivalent control-flow bodies must use braces, including single-statement bodies.
- Put braced bodies on separate lines.
- Prefer guard clauses when they reduce unnecessary nesting.
- Deep nesting is a signal to examine responsibility and control flow, not a fixed numerical violation.
- Do not mechanically split methods solely to satisfy a line-count or nesting threshold.
- Avoid regular expressions by default.
- Do not use regular expressions to parse structured formats when a structured parser or typed API exists.
- Prefer explicit parsing, typed APIs, string scanning, or a small readable state machine.
- Use a regular expression only when it is clearly simpler for a bounded matching problem.
- Non-trivial regular expressions must have clearly understood matching boundaries and must avoid unnecessary performance risks.
- Dirty or inherently unstructured external input, including unreliable machine-generated text, may use regular expressions as a controlled parsing fallback.

### Method Parameters

When a method has more than roughly four or five parameters, examine whether its responsibility or API shape should be improved.

- First determine whether the method owns too many responsibilities.
- Introduce a parameter object only when the values form a coherent business concept such as `TransferRequest`, `PricingContext`, or `RecallQuery`.
- Do not mechanically wrap unrelated values in containers such as `FooParams` or `FooArgs`.
- If the values have no meaningful relationship beyond being passed to the same method, prefer examining whether the method should be split.
- External protocol request and response objects are exempt because their shape is defined by the external contract.

## 3. Types, Data, And Boundaries

### Closed Values And Domain Types

- Stable closed value sets such as error codes, statuses, modes, categories, strategies, and types must use an enum or an equivalent closed type representation.
- Do not compare internal business state through raw string or numeric literals such as `status.equals("ACTIVE")`.
- Convert external string or numeric representations to the closed type once at the system boundary.
- Internal business logic must operate on the typed value rather than repeatedly parsing the raw representation.
- Unknown external values must have an explicit handling strategy such as rejection, logging plus fallback, or a documented unknown state. Do not silently ignore or discard them.
- Use value objects when they protect invariants, group semantically inseparable values, or remove ambiguity from important primitives.
- Do not mechanically wrap every primitive.
- Do not represent business state with `null`, zero, empty strings, arbitrary constants, or other valid-looking sentinel values.

### Typed Domain Boundaries

- Business data passed across modules or architectural boundaries must use named, typed domain objects when the data has a stable business shape.
- Do not pass raw JSON strings, `Map<String, Object>`, or generic `Object` values through business modules to carry known business semantics.
- Parse external representations once at the boundary, operate on typed objects internally, and serialize once at the output boundary.
- Do not repeatedly parse and serialize the same business data through intermediate layers.
- Dynamic maps and JSON nodes are allowed inside protocol adapters, external dirty-input parsers, generic data infrastructure, and inherently dynamic integrations.
- Once stable business data has been parsed successfully, convert it to a typed representation before exposing it outside the boundary implementation.
- Open row structures from generic data sources such as arbitrary SQL query results may remain dynamic when the structure is intentionally open.
- Weakly typed parameter containers may remain internal to LLM, plan, or similar inherently dynamic parsers.

### JSON Processing

- A project should use one standard JSON stack rather than mixing unrelated JSON libraries throughout business code.
- When Java-specific behavior is involved, follow the JSON requirements in `java-guidelines.md`.
- Prefer binding stable JSON structures to named types rather than repeatedly navigating dynamic fields.
- Dynamic JSON representations belong at boundaries and parser internals when they are genuinely required.
- After a typed representation replaces text-based or dynamic JSON handling, remove obsolete string-processing hacks and related historical comments.

### Absence And State

- Methods that may legitimately return no value should expose that possibility through the language or project's established nullability or optional-value mechanism.
- Collection-returning APIs must return an empty collection rather than `null`.
- Do not add null handling that contradicts a verified non-null contract.
- Do not use valid-looking domain values as substitutes for absence, failure, unsupported state, or fallback behavior.
- If optional-value wrappers are used, follow the language-specific rules for where they are appropriate.

### Persistence And Data Boundaries

- Persistence entities, DAOs, repositories, query models, and persistence-vendor types belong to their owning persistence boundary.
- Cross-module code should depend on domain models, read models, commands, or role interfaces rather than another module's persistence internals.
- Do not expose ORM, database, cache, or storage implementation details through core APIs unless they are explicitly part of the contract.
- Schema design must follow actual ownership, lifecycle, integrity, query, indexing, update, concurrency, and retention requirements.
- Do not normalize, denormalize, flatten, split, merge, or move data into document payloads mechanically.
- Every structural persistence change must have a concrete access-pattern, integrity, lifecycle, or performance justification.

## 4. Exceptions, Logging, And Security

### Exceptions

- Business exceptions must carry enough context to identify the failed operation, relevant entity identifiers, and meaningful failure reason.
- Use exceptions for exceptional failures, not normal business branching.
- Catch an exception only when the current layer has a concrete responsibility to recover, translate, enrich, compensate, or record it.
- A caught exception must be handled, recorded appropriately, or rethrown. Never silently swallow an exception.
- Preserve the original cause when wrapping an exception.
- Do not add broad catch blocks merely to make code appear defensive.

### Logging

- Use the project's logging framework. Do not use ad hoc printing as application logging.
- `ERROR` is for failures that require intervention or represent an unsuccessful operation that cannot recover at the current level.
- `WARN` is for abnormal conditions that can be recovered or tolerated automatically.
- `INFO` records meaningful business or lifecycle events.
- `DEBUG` is for development and diagnostic detail.
- Important `ERROR` and `WARN` logs must contain enough context to investigate the problem, such as relevant business identifiers, key input information, failure reasons, and exception details.
- Core business paths such as payment, order creation, or important state transitions should record enough meaningful before-and-after context to reconstruct failures when the project logging model supports it.
- Avoid noisy logging on high-frequency non-critical paths.
- Preserve trace IDs, request IDs, correlation IDs, or equivalent identifiers when the architecture provides them.
- Do not log credentials, secrets, tokens, passwords, or unnecessarily sensitive data.
- Do not use console printing or direct stack-trace printing as a substitute for structured logging.

### Security And External Input

- Treat external input as untrusted unless a verified trusted-boundary contract proves otherwise.
- Validate HTTP parameters, uploaded content, third-party API responses, messages, and other external data according to the actual contract before relying on them.
- Use parameterized queries, ORM APIs, or equivalent safe mechanisms. Do not construct executable SQL from untrusted input through string concatenation.
- Prefer environment variables, configuration systems, or secret-management infrastructure for secrets, tokens, passwords, and credentials.
- If a framework limitation or temporary local debugging requirement genuinely requires a hard-coded sensitive value, document the reason and scope and ensure it does not remain in a production branch or release.
- User-facing errors must not expose stack traces, SQL statements, server paths, credentials, or unnecessary implementation details.

## 5. Resources, Concurrency, And Performance

- Files, streams, network connections, database connections, locks, and similar resources must be released deterministically using the language or framework's ownership mechanism.
- Do not rely on garbage collection to release resources.
- External HTTP, RPC, database, or similar calls must have appropriate timeout behavior.
- Retry only failures known to be retryable and only when a concrete retry strategy exists.
- Do not retry non-idempotent operations without an explicit safeguard.
- Operations that may be retried, redelivered, or executed more than once must provide an idempotency mechanism when duplicate execution could cause incorrect behavior.
- Avoid unnecessary shared mutable state.
- When shared mutable state is required, use an explicit concurrency strategy appropriate to the runtime and data model.
- Do not assume single-threaded, ordered, or exactly-once execution without evidence from the actual execution model.
- Avoid obviously inefficient behavior such as repeated I/O, repeated parsing or serialization, N+1 access patterns, unnecessary scans inside loops, unbounded accumulation, or avoidable high-complexity algorithms.
- Choose data structures and algorithms appropriate to the actual workload.
- Do not introduce complex optimization without evidence that the expected workload requires it.

## 6. Testing And Verification

- Code changes must be verified with relevant automated tests whenever the project provides a runnable test path.
- New business behavior must include corresponding tests.
- Bug fixes should include a regression test reproducing the original failure when practical.
- Tests must be independent and must not depend on execution order or shared mutable state.
- Use mocks or test doubles to isolate genuine external dependencies such as databases, networks, third-party services, clocks, or nondeterministic infrastructure.
- Do not mock the core logic of the unit being tested.
- Assert externally meaningful behavior rather than unnecessary private implementation details.
- Do not over-specify call counts, private state, or internal sequencing unless those details are part of the behavior being verified.
- Do not weaken a test merely to make the current implementation pass.
- When a test fails after a change, first determine whether the production code is wrong, the test is wrong, or the requirement intentionally changed.
- Without evidence that the contract changed, do not delete assertions, relax expected results, add meaningless mocks, skip tests, or disable coverage to accommodate the implementation.
- Run focused tests first, then broader regression or build checks according to the affected scope.
- A successful code review or static inspection does not replace runnable verification when runnable verification is available.

## 7. Code Smells And Scope Boundaries

Code smells are investigation signals. Their presence does not automatically authorize refactoring, especially outside the current task scope.

| Smell | Description |
|---|---|
| Type guessing | Inferring a type from string prefixes, substrings, naming patterns, or other indirect signals instead of using an explicit type representation. |
| Regex overreach | Extracting structured business data with regular expressions when a structured parser or typed representation should own the problem. |
| Repeated downcasting | Repeated `instanceof`, subtype probing, or downcasting against an abstraction instead of using its contract. |
| Dual-key fallback | Reading the same semantic value from multiple keys to support compatibility that has not been demonstrated as necessary. |
| Sentinel value | Using `null`, zero, empty strings, arbitrary constants, or otherwise valid-looking values to represent absence, failure, or hidden state. |
| Repeated switch | Maintaining the same enum or type branching logic in multiple places, causing one business rule to have multiple owners. |
| Dead code | Code with no meaningful execution path or code made obsolete by the current implementation. |
| Parallel replacement | Introducing `XxxV2`, `newXxx()`, `calculateImpl2()`, or similar parallel implementations instead of replacing obsolete behavior, leaving multiple versions of the same capability behind. |
| God class | A class containing multiple unrelated responsibilities and multiple independent reasons to change. |
| Nullable-field union | Modeling mutually exclusive states with multiple nullable fields, allowing invalid combinations such as all fields being absent or multiple alternatives being present simultaneously. |
| Shared mutable state | Mutable state accessed concurrently without a clear ownership or synchronization strategy. |
| Feature envy | Logic that primarily operates on another object's data and likely belongs with that object's responsibility. |
| Data clump | The same semantically related group of values repeatedly traveling together without a meaningful domain representation. |
| Primitive obsession | Repeated use of primitives for concepts such as money, currency, identifiers, or other values whose invariants or semantics justify a domain type. |
| Long parameter list | A large parameter list caused by mixed responsibilities or an unmodeled coherent domain concept. |
| Flag argument | A boolean or mode parameter that switches a method between substantially different responsibilities. |
| Divergent change | One component changes for multiple unrelated reasons, indicating mixed responsibilities. |
| Shotgun surgery | One logical change requires unnecessary edits across many components because responsibility is scattered. |
| Middleman | A component that only forwards calls and provides no meaningful responsibility, boundary, policy, or abstraction. |
| Swallowed exception | A caught failure that is neither handled, appropriately recorded, nor rethrown. |
| Exception-driven control flow | Using exceptions to represent expected business alternatives. |
| Magic number | A business-significant numeric literal whose meaning is not expressed by a named constant, enum, or domain type. |
| Deep nesting | Control flow whose nesting materially increases cognitive complexity and can be clarified through guards, decomposition, or responsibility extraction. |
| Exposed mutable collection | Returning an internal mutable collection in a way that allows callers to bypass the owning object's invariants. |
| Circular dependency | Modules or components depending on each other in both directions, indicating unclear responsibility boundaries. |
| Anemic domain model | Domain data with meaningful behavior whose rules are instead scattered through unrelated external services or utilities. |
| Copy-pasted near-duplicate logic | Multiple implementations representing the same behavior with only minor parameter or structural differences. |
| Test accommodation | Weakening assertions, changing expected behavior without evidence, adding meaningless mocks, or disabling tests merely to make an incorrect implementation pass. |
| Speculative abstraction | Introducing an interface, strategy, factory, extension point, configuration option, or hierarchy without a current variation point or consumer. |
| Speculative compatibility | Adding fallback keys, aliases, dual reads, legacy branches, or other compatibility behavior without evidence of an actual compatibility requirement. |
| Speculative defense | Adding null checks, catch blocks, retries, fallback values, or validation for states that the verified contract does not permit or that have no concrete handling strategy. |
| Public API leakage | Exposing implementation details or widening visibility without an actual external or cross-module consumer. |

Before refactoring a smell, identify the concrete problem it causes.

Do not fix unrelated smells during a focused task.

### Boundary Exceptions

The exceptions in this section apply only to data-modeling and representation rules.

Rules requiring enums or closed types, typed domain objects, value objects, typed JSON representations, or avoidance of dynamic structures primarily constrain module and architectural boundaries.

Protocol adapters, external-input parsers, generic data-access infrastructure, LLM or plan parsers, and inherently dynamic integrations may reasonably use maps, dynamic JSON nodes, weakly typed parameter containers, or regular expressions internally when those representations fit the boundary.

External protocols may also require DTO or payload shapes that would not otherwise be chosen as internal domain models.

Once stable business data leaves such a boundary, convert it to the appropriate typed representation.

These exceptions do not exempt any layer from:

- correctness and verification requirements;
- testing requirements;
- exception-handling requirements;
- logging requirements;
- security and input-validation requirements;
- resource-management requirements;
- concurrency requirements.

Do not refactor code merely because a `Map`, dynamic JSON node, regular expression, or weakly typed structure appears inside a boundary where its use is explicitly permitted.

Before changing such code, first establish that the representation actually violates a rule applicable to that layer.
