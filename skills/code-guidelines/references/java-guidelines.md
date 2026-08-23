# Java And JVM Guidelines

These requirements apply to Java, Kotlin, and related JVM code. They use the normative model defined by `../SKILL.md` and supplement `engineering-guidelines.md`.

## 1. Java Code And Types

### Code Style

- Types MUST be imported and referenced by simple name when an unambiguous normal import is possible.

**EXCEPTION:** A fully qualified type name MAY be used when required to disambiguate two necessary types with the same simple name or when the language or generated context requires it.

- `if`, `for`, and `while` bodies MUST use braces, including single-statement bodies.
- Braced bodies MUST use multi-line form.
- Method bodies MUST use multi-line form. One-line methods such as `public ID id() { return id; }` MUST NOT be introduced, including accessors, factories, and anonymous-class implementations.
- Repository formatter, compiler, and static-analysis configuration MUST be followed.
- `@author` documentation MUST NOT be introduced as a universal convention.

**TRIGGER:** The repository has an explicit and current author-tag convention.

- Existing author-tag conventions SHOULD be followed within their verified scope.

### Nullability And Optional Values

- One nullability annotation system SHOULD be used consistently across a project.

**TRIGGER:** JSpecify is the established nullability system.

- `@NullMarked` SHOULD be applied at package level when that matches the project structure.
- Nullable exceptions MUST be marked with `@Nullable` according to the established contract.

- A method that may legitimately return no value MUST expose that possibility through the established nullability contract.
- Collection-returning methods MUST return empty collections instead of `null`.
- `Optional<T>` MAY be used when it improves return-value semantics.

**TRIGGER:** `Optional<T>` is used.

- It MUST be used only as a method return type.
- It MUST NOT be used as a field type, method parameter, or collection element.

- Redundant null checks MUST NOT be added when an established non-null contract guarantees the value.

### Enums And Domain Types

- Stable closed sets such as error codes, statuses, modes, categories, strategies, and types MUST use enums or an equivalent closed JVM representation.
- External string or numeric values MUST be converted to the enum once at the system boundary.
- Internal business logic MUST pass the enum value rather than repeatedly interpreting the raw representation.
- Unknown external enum values MUST have an explicit handling strategy.
- An enum MUST NOT be used when the value set is intentionally open or externally extensible.
- Immutable value objects SHOULD represent structured values with meaningful invariants or semantics and no independent identity.
- Arrays, maps, or unrelated primitives SHOULD NOT be used to assemble a stable domain concept when a small value object materially improves correctness and clarity.

### Method Parameters And Collections

**TRIGGER:** A method has more than five parameters or repeatedly receives the same group of values.

- Excessive method responsibility SHOULD be investigated first.
- A request, command, context, or value object SHOULD be introduced only when the parameters form a coherent domain concept.
- Generic containers such as `FooParams` or `FooArgs` MUST NOT be created merely to reduce visible parameter count.

- State SHOULD be immutable unless mutation is required by the object's responsibility.
- Mutable state SHOULD be kept as narrowly scoped as practical.
- Internal mutable collections MUST NOT be exposed directly when external mutation would bypass invariants.
- Defensive copies, immutable collections, or encapsulated operations SHOULD be used when callers must not mutate internal state.
- Code MUST NOT rely on iteration order from a collection implementation that does not guarantee it.

## 2. Object Design And Dependencies

- Constructor injection SHOULD be used for required dependencies.
- Required collaborators SHOULD be complete when an object is constructed.
- Field injection MUST NOT be used for required dependencies.
- Setter injection SHOULD NOT be used for required dependencies.

**EXCEPTION:** Setter or framework-managed injection MAY be used when a verified framework or repository constraint requires it and constructor injection is not viable.

- An interface MUST NOT be introduced solely because a concrete class is injected.

**TRIGGER:** A new interface is introduced.

- It MUST represent a real role, architectural boundary, substitutable implementation, or verified project convention.
- A single implementation plus hypothetical future extensibility MUST NOT by itself justify the interface.

- A `BaseEntity` or similar superclass MUST NOT be introduced merely to deduplicate fields.
- Composition SHOULD be preferred over implementation inheritance.
- Concrete implementations MUST NOT be inherited from solely to reuse behavior.
- Domain behavior SHOULD remain independent from framework infrastructure when a meaningful domain boundary exists.
- `Service` or `Manager` SHOULD NOT be used as a role name when a more precise responsibility can be named.

**TRIGGER:** The repository explicitly establishes an interface plus `Impl` convention for Spring bean contracts.

- That convention SHOULD be followed within the verified repository scope.

- Persistence, cache, upstream, or fallback implementation details SHOULD NOT be exposed through core domain contracts.
- Domain and value objects SHOULD be immutable by default. Mutation MAY be used when state change is part of the object's actual responsibility.

## 3. JSON, Time, And Persistence

### JSON

- The project's established JSON stack MUST be used consistently.

**TRIGGER:** No conflicting project convention exists and Jackson is already available as the standard JSON stack.

- Jackson SHOULD be used for new JSON behavior.

- Jackson, Gson, `org.json`, or unrelated JSON models SHOULD NOT be mixed throughout business code.

**TRIGGER:** A third-party SDK requires a different JSON representation.

- The foreign JSON dependency SHOULD remain inside the SDK adapter.
- Data leaving the adapter MUST be converted back to the project's standard representation or typed domain model.

- Stable structured JSON SHOULD be bound to named types instead of repeatedly navigating dynamic nodes through chains such as `json.get("a").get("b").asText()`.
- JSON SHOULD be parsed once at the boundary and typed objects SHOULD be passed through internal business logic.
- The same payload SHOULD NOT be repeatedly parsed and serialized between internal layers.

**EXCEPTION:** Dynamic JSON nodes and maps MAY remain internal to protocol adapters, dirty-input parsers, generic data-source adapters, and inherently dynamic integrations.

Once stable business data leaves such a boundary, it MUST be converted to a typed representation.

- Application-configured serializers or the established project JSON abstraction SHOULD be reused rather than creating independent serializer configurations throughout business code.
- Obsolete parsing hacks and their related comments MUST be removed when typed parsing replaces them.

### Date And Time

- New date and time code MUST use `java.time`.
- New uses of `java.util.Date` or `Calendar` MUST NOT be introduced.

**EXCEPTION:** Legacy date types MAY be used at a required compatibility boundary, but SHOULD be converted to `java.time` immediately after crossing that boundary.

- `Instant` SHOULD be used for UTC timeline timestamps when appropriate to the contract.
- `LocalDate` MUST be used for date-only business concepts.
- `LocalDateTime` MUST be used only when a date and time intentionally have no zone or offset semantics.
- `ZonedDateTime`, `OffsetDateTime`, or an explicit `ZoneId` SHOULD be used when zone semantics are part of the value or operation.
- Cross-time-zone storage and transport MUST carry explicit time semantics.
- Cross-time-zone business behavior MUST NOT depend implicitly on the server default time zone.
- Project-standard date and time formatting or parsing utilities SHOULD be reused when they exist.
- Independent `DateTimeFormatter` construction SHOULD NOT be scattered through business code when the project provides a standard abstraction.

### Persistence

- Entities, DAOs, repositories, query models, and persistence-specific implementation details SHOULD remain inside the owning feature's persistence boundary.
- JPA, Hibernate, Spring Data, JDBC, and other persistence-vendor types SHOULD NOT appear in core APIs unless repository architecture explicitly defines them as part of the contract.
- JPA entities SHOULD NOT be exposed directly as cross-module domain contracts or public API models.
- Cross-feature code SHOULD depend on role interfaces, domain models, commands, or read models rather than another feature's repository or persistence entity.
- Parameterized ORM, query, or JDBC APIs MUST be used for untrusted query values.
- Executable SQL MUST NOT be built by concatenating untrusted values.
- Hidden lazy-loading dependencies SHOULD NOT cross architectural boundaries.
- Transaction boundaries MUST follow verified business consistency requirements.
- A shared entity superclass MUST NOT be introduced merely to remove repeated identity or audit fields.
- Repository-specific identity strategies, schema naming, auditing conventions, timestamp representations, and repository patterns MUST be established in project-level guidelines rather than inferred as universal JVM rules.

## 4. Kotlin And Java Interoperability

Kotlin in mixed Java and Kotlin codebases MUST use a restrained, Java-oriented style that preserves predictable interoperability and structural consistency.

- Existing common and configuration Kotlin sources SHOULD remain in Kotlin unless a verified redesign requires otherwise.
- A Kotlin file MUST declare at most one top-level class.
- Top-level Kotlin functions MUST NOT be introduced.
- Top-level Kotlin properties MUST NOT be introduced.
- Methods and properties MUST belong to a role-appropriate `class` or `object`.
- Factory functions SHOULD reside in a `companion object`.
- Local and nested classes MAY be used when they are implementation details scoped similarly to their Java equivalents.
- Kotlin source files MUST be stored under Java source directories such as `src/main/java` and `src/test/java`.
- Java interoperability MUST remain straightforward.
- Kotlin-specific API shapes SHOULD NOT be introduced solely for idiomatic style when they materially complicate Java callers.
- Explicit nullability MUST be preserved across Java and Kotlin boundaries.

**TRIGGER:** Java APIs use JSpecify.

- Kotlin code MUST respect the resulting nullability contracts.

- KDoc MUST use standard multi-line documentation blocks. Single-line KDoc MUST NOT be used.
- Existing Java MUST NOT be rewritten to Kotlin, and existing Kotlin MUST NOT be rewritten to Java, merely because of language preference.
- Kotlin constructs that obscure control flow, ownership, mutation, or Java-visible API behavior SHOULD NOT be used when a simpler Java-oriented form is clearer.
- Top-level DSLs, extension-heavy APIs, operator-heavy APIs, or Kotlin-only abstractions MUST NOT be introduced without a concrete requirement that justifies their interoperability and maintenance cost.
- Explicit classes and methods SHOULD be preferred when Kotlin language features would make behavior materially less obvious to Java-oriented maintainers.

## 5. Exceptions, Logging, Resources, And Tests

### Exceptions And Logging

- Specific exception types SHOULD communicate failure semantics.
- Business exceptions MUST include enough context to identify the failed operation, relevant entity identifiers, and a meaningful failure reason.
- The original cause MUST be preserved when wrapping exceptions.
- `Exception`, `RuntimeException`, or similarly broad types SHOULD NOT be caught.

**EXCEPTION:** A broad catch MAY be used at a boundary that genuinely owns broad failure translation, containment, or recovery.

- Exceptions MUST NOT be used for expected business alternatives.
- A caught exception MUST be handled, logged appropriately, or rethrown.
- `printStackTrace()` MUST NOT be used.
- The project's logging framework MUST be used for application logging.
- `System.out.println` and `System.err.println` MUST NOT be used for application or committed debugging logs.
- Parameterized logging SHOULD be used when supported by the logging framework.
- Diagnostic logs SHOULD include relevant domain and operation context.
- Trace or request identifiers SHOULD be preserved when the project provides them.
- Secrets and unnecessarily sensitive values MUST NOT be logged.

### Resources And External Calls

- Owned `AutoCloseable` resources MUST use try-with-resources or an equivalent deterministic ownership mechanism.
- Resources whose lifecycle is owned by the framework or container MUST NOT be manually closed.
- HTTP, RPC, database, and similar external calls MUST have bounded timeout behavior appropriate to their contract.

**TRIGGER:** Automatic retries are introduced for an external call.

- The failure MUST be established as retryable.
- Side-effecting operations MUST be idempotent or protected by an explicit duplicate-execution mechanism.

### Testing And Build

- The repository's established Java and Kotlin test frameworks and conventions MUST be followed.
- Tests MUST remain independent.
- Tests SHOULD assert behavior rather than unnecessary implementation details.
- External or infrastructure boundaries MAY be mocked when isolation is required.
- Core behavior under test MUST NOT be mocked away.
- Bug fixes SHOULD add regression coverage when a practical test path exists.
- New business behavior MUST have corresponding tests.
- Assertions or expected behavior MUST NOT be weakened merely to accommodate a failing implementation.
- The repository's existing build system, formatter, compiler checks, and static-analysis tools MUST be respected.
- Build configuration MUST be treated as production code and changes MUST remain focused.
- Unrelated formatting, plugins, dependencies, or build configuration MUST NOT be modified during a focused change.
- Unfamiliar build-tool, framework, or library APIs MUST be verified before introduction or modification.
