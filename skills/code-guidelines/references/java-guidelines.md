# Java And JVM Guidelines

These requirements apply to Java, Kotlin, and related JVM code unless a repository-specific rule explicitly overrides them.

Apply the general engineering requirements in `engineering-guidelines.md` in addition to this document.

## 1. Java Code And Types

### Code Style

- Import types and use their simple names. Do not use fully qualified type references in code bodies when a normal import is possible.
- Use braces for all `if`, `for`, and `while` bodies, including single-statement bodies.
- Put braced bodies on separate lines.
- Write every method body across multiple lines. Do not use one-line methods such as `public ID id() { return id; }`, including accessors, factories, and anonymous-class implementations.
- Follow the repository formatter, compiler, and static-analysis configuration.
- Add `@author` documentation only when explicitly required by the repository. Do not introduce author tags as a universal Java convention.

### Nullability And Optional Values

- Use one nullability annotation system consistently across the project.
- When JSpecify is used, prefer package-level `@NullMarked` and annotate nullable exceptions with `@Nullable`.
- Methods that may legitimately return no value must expose that possibility through the established nullability contract.
- Collection-returning methods must return empty collections instead of `null`.
- `Optional<T>` is recommended when it improves return-value semantics, but it is not mandatory.
- If `Optional<T>` is used, use it only as a method return type.
- Do not use `Optional<T>` as a field type, method parameter, or collection element.
- Do not add redundant null checks when an established non-null contract guarantees the value.

### Enums And Domain Types

- Use enums for stable closed sets such as error codes, statuses, modes, categories, strategies, and types.
- Convert external string or numeric values to enums once at the system boundary.
- Internal business logic must pass enum instances rather than repeatedly interpreting raw values.
- Unknown external enum values must have an explicit handling strategy.
- Do not use an enum when the value set is intentionally open or externally extensible.
- Prefer immutable value objects for structured values that have no identity but carry meaningful invariants or semantics.
- Do not use arrays, maps, or unrelated primitives to temporarily assemble a stable domain concept when a small value object materially improves correctness and clarity.

### Method Parameters And Collections

- When a method has more than roughly four or five parameters, first determine whether the method owns too many responsibilities.
- Introduce a request, command, context, or value object only when the parameters form a coherent domain concept.
- Do not create generic `FooParams` or `FooArgs` containers merely to reduce parameter count.
- Prefer immutable state unless mutation is required by the object's responsibility.
- Keep mutable state as narrowly scoped as practical.
- Do not expose internal mutable collections directly.
- Use defensive copies, immutable collections, or encapsulated operations when callers must not mutate internal state.
- Do not rely on iteration order from a collection implementation that does not guarantee it.

## 2. Object Design And Dependencies

- Prefer constructor injection for required dependencies.
- Required collaborators should be complete when an object is constructed.
- Do not use field injection for required dependencies.
- Do not use setter injection for required dependencies unless a framework or verified project requirement makes it necessary.
- Do not create an interface solely because a concrete class is injected.
- Introduce an interface when it represents a real role, architectural boundary, substitutable implementation, or explicit project convention.
- When only one implementation exists and no real boundary or second implementation is required, prefer the concrete type over a speculative interface.
- Do not create a `BaseEntity` or similar superclass merely to deduplicate fields.
- Prefer composition over implementation inheritance.
- Do not inherit from concrete implementations solely to reuse behavior.
- Keep domain behavior independent from framework infrastructure when a meaningful domain boundary exists.
- Do not create a `Service` or `Manager` name when a more precise role name exists.
- When the repository explicitly establishes an interface plus `Impl` convention for Spring bean contracts, follow that project convention rather than treating it as a universal Java rule.
- Do not expose persistence, cache, upstream, or fallback implementation details through core domain contracts.
- Prefer immutable domain and value objects by default. Use mutable objects only when mutation is part of the object's actual responsibility.

## 3. JSON, Time, And Persistence

### JSON

- Use the project's established JSON stack consistently.
- When no conflicting project convention exists and Jackson is available, prefer Jackson as the standard JSON library.
- Do not mix Jackson, Gson, `org.json`, or other JSON models throughout business code.
- If a third-party SDK requires another JSON representation, keep that dependency inside the SDK adapter and convert at the boundary.
- Bind stable structured JSON to named types instead of repeatedly navigating dynamic nodes through chains such as `json.get("a").get("b").asText()`.
- Parse JSON once at the boundary and pass typed objects through internal business logic.
- Do not repeatedly parse and serialize the same payload between internal layers.
- Dynamic JSON nodes and maps are allowed inside protocol adapters, external dirty-input parsers, generic data-source adapters, and inherently dynamic integrations when appropriate.
- Once stable business data has been parsed, convert it to a typed representation before exposing it outside the adapter or parser.
- Use the application's configured serializer or project JSON abstraction rather than creating independent serializer configurations throughout business code.
- After replacing text-based or dynamic JSON processing with typed parsing, remove obsolete parsing hacks and their related comments.

### Date And Time

- Use `java.time` for new date and time code.
- Do not introduce new uses of `java.util.Date` or `Calendar` except at a required compatibility boundary.
- Use `Instant` for UTC timeline timestamps when appropriate.
- Use `LocalDate` for date-only business concepts.
- Use `LocalDateTime` only when a date and time intentionally have no time-zone or offset semantics.
- Use `ZonedDateTime`, `OffsetDateTime`, or an explicit `ZoneId` when time-zone semantics are part of the value or operation.
- Cross-time-zone storage and transport must carry explicit time semantics.
- Do not implicitly depend on the server default time zone for cross-time-zone business behavior.
- Use project-standard date and time formatting and parsing utilities when they exist.
- Do not scatter independent `DateTimeFormatter` construction throughout business code when the project provides a standard abstraction.

### Persistence

- Keep entities, DAOs, repositories, query models, and persistence-specific implementation details inside the owning feature's persistence boundary.
- Keep JPA, Hibernate, Spring Data, JDBC, and other persistence-vendor types out of core APIs unless the repository architecture explicitly defines otherwise.
- Do not expose JPA entities as cross-module domain contracts or public API models by default.
- Cross-feature code must depend on role interfaces, domain models, commands, or read models rather than another feature's repository or persistence entity.
- Prefer parameterized ORM, query, or JDBC APIs.
- Do not build executable SQL by concatenating untrusted values.
- Avoid hidden lazy-loading dependencies across architectural boundaries.
- Transaction boundaries must follow actual business consistency requirements.
- Do not introduce a shared entity superclass merely to remove repeated identity or audit fields.
- Repository-specific identity strategies, schema naming, auditing conventions, timestamp representations, and repository patterns belong in project-level guidelines.

## 4. Kotlin And Java Interoperability

Java and Kotlin are both allowed, but Kotlin must use a restrained, Java-oriented style in mixed Java and Kotlin codebases.

Prioritize predictable Java interoperability, structural consistency, and long-term maintainability across both languages.

- Keep existing common and configuration Kotlin sources in Kotlin unless a redesign explicitly requires otherwise.
- Declare at most one top-level class per Kotlin file.
- Do not declare top-level Kotlin functions.
- Do not declare top-level Kotlin properties.
- Put methods and properties inside a role-appropriate `class` or `object`.
- Put factory functions in a `companion object`.
- Local and nested classes are allowed when they are implementation details scoped similarly to their Java equivalents.
- Store Kotlin source files under Java source directories such as `src/main/java` and `src/test/java`.
- Preserve straightforward Java interoperability.
- Do not introduce Kotlin-specific API shapes solely for idiomatic style when they materially complicate Java callers.
- Preserve explicit nullability across Java and Kotlin boundaries.
- When Java APIs use JSpecify, Kotlin code must respect the resulting nullability contracts.
- KDoc must use standard multi-line documentation blocks. Do not use single-line KDoc.
- Do not rewrite existing Java to Kotlin or existing Kotlin to Java merely because of language preference.
- Avoid Kotlin constructs that obscure control flow, ownership, mutation, or Java-visible API behavior when a simpler Java-oriented form is clearer.
- Do not introduce top-level DSLs, extension-heavy APIs, operator-heavy APIs, or Kotlin-only abstractions without a concrete requirement that justifies their interoperability and maintenance cost.
- Prefer explicit classes and methods over Kotlin language features that make the same behavior substantially less obvious to Java-oriented maintainers.

## 5. Exceptions, Logging, Resources, And Tests

### Exceptions And Logging

- Use specific exception types that communicate failure semantics.
- Business exceptions must include enough context to identify the failed operation, relevant entity identifiers, and meaningful failure reason.
- Preserve the original cause when wrapping exceptions.
- Do not catch `Exception`, `RuntimeException`, or similarly broad types unless the current boundary genuinely owns broad failure translation or recovery.
- Do not use exceptions for expected business alternatives.
- A caught exception must be handled, logged appropriately, or rethrown.
- Do not call `printStackTrace()`.
- Use the project's logging framework.
- Do not use `System.out.println` or `System.err.println` for application or committed debugging logs.
- Use parameterized logging when supported.
- Include relevant domain and operation context in diagnostic logs.
- Preserve trace or request identifiers when the project provides them.
- Do not log secrets or unnecessarily sensitive values.

### Resources And External Calls

- Use try-with-resources for owned `AutoCloseable` resources.
- Do not manually close resources whose lifecycle is owned by the framework or container.
- HTTP, RPC, database, and similar external calls must have appropriate timeout behavior.
- Retry only failures known to be retryable and only when a concrete retry strategy exists.
- Retryable operations that can cause side effects must be idempotent or protected by an explicit idempotency mechanism.

### Testing And Build

- Follow the repository's established Java and Kotlin test frameworks and conventions.
- Keep tests independent.
- Assert behavior rather than unnecessary implementation details.
- Mock external or infrastructure boundaries when isolation is required.
- Do not mock the core behavior being tested.
- Add regression coverage for bug fixes when practical.
- Add tests for new business behavior.
- Do not weaken assertions or modify expected behavior merely to accommodate a failing implementation.
- Use the repository's existing build system, formatter, compiler checks, and static-analysis tools.
- Treat build configuration as production code and keep changes focused.
- Do not modify unrelated formatting, plugins, dependencies, or build configuration while solving a focused task.
- Verify unfamiliar build-tool, framework, or library APIs before introducing or modifying their usage.
