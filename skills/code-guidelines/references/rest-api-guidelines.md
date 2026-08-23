# HTTP API Guidelines

These requirements apply to externally consumed HTTP APIs and internal HTTP APIs whose contracts are shared across components. They use the normative model defined by `../SKILL.md`.

The goal is to design stable, explicit, secure, and evolvable HTTP contracts without forcing domain behavior into artificial CRUD shapes.

## 1. API Contract Principles

Public and shared APIs MUST be treated as explicit contracts. Implementation structure MUST NOT define an external contract accidentally.

API design SHOULD prioritize stable semantics, explicit ownership, predictable behavior, compatibility, security boundaries, and operational clarity. Internal framework, persistence, and service implementation details SHOULD NOT leak into API contracts.

An API contract includes externally observable behavior such as URI structure, HTTP methods, request and response fields, status codes, error semantics, authentication and authorization requirements, documented side effects, ordering, pagination, and concurrency behavior.

Existing implementation behavior MUST NOT be treated as contractual solely because it currently exists. A compatibility requirement MUST be supported by a verified consumer, persisted representation, published contract, or explicit requirement.

API semantics MUST be explicit. Clients MUST NOT be required to infer material behavior from undocumented implementation details, sentinel values, omitted fields, or incidental response shapes.

## 2. Resource Modeling

APIs SHOULD model stable business concepts as resources when those concepts have meaningful identity, lifecycle, state, ownership, or relationships.

A resource MUST NOT be introduced merely to mirror a database table, framework class, controller, or service method. API resource structure and persistence structure MAY differ when they represent different responsibilities.

Resource ownership and containment SHOULD be represented in paths when they are part of the resource's identity or authorization boundary. Artificial nesting MUST NOT be introduced only to make a URI appear more RESTful.

A resource MUST have a clear external identity. Internal database identifiers SHOULD NOT become public identifiers only because they are convenient. Multiple identifiers MAY exist when they represent distinct business concepts or verified integration requirements.

A compatibility alias or secondary identifier MUST NOT be introduced for hypothetical consumers.

## 3. URI And HTTP Method Semantics

API paths SHOULD represent domain resources or meaningful domain operations rather than implementation details.

Names such as controller names, service names, repository names, database table names, and generic operation wrappers SHOULD NOT appear in public paths unless they are themselves verified domain concepts.

HTTP methods MUST preserve their standard semantics.

### GET

GET MUST NOT intentionally perform business state changes. A GET request MUST NOT execute payments, create resources, perform irreversible transitions, or trigger equivalent side effects.

Incidental operational effects such as access logging, metrics, or cache population MAY occur when they do not change the requested business state.

### POST

POST MAY be used to create a server-identified resource, submit a domain command, or perform an operation that does not have PUT or PATCH semantics.

POST behavior MUST be explicit when the operation is not ordinary resource creation.

### PUT

**TRIGGER:** The client provides the intended complete representation of a resource at a known URI.

PUT SHOULD represent replacement semantics and MUST NOT silently behave as an undocumented partial update.

### PATCH

**TRIGGER:** The client modifies only part of an existing resource.

PATCH SHOULD be used for partial modification. The patch representation MUST define the meaning of omitted fields, explicit null values, empty values, and removal or reset operations when those states are possible.

Clients MUST NOT be required to guess whether an omitted field means unchanged, cleared, removed, defaulted, or invalid.

### DELETE

DELETE SHOULD represent removal of the addressed resource or the resource's externally visible existence. Logical deletion MAY satisfy DELETE semantics when retention requirements require the underlying data to remain stored.

Deletion behavior MUST be explicit when restoration, retention, asynchronous deletion, or dependent-resource behavior materially affects consumers.

### Custom Actions

**TRIGGER:** A meaningful domain operation does not map cleanly to standard resource creation, retrieval, replacement, partial modification, or deletion.

A custom action MAY be introduced. The action MUST represent a domain operation rather than a generic wrapper around an internal service method. Its input, output, authorization, side effects, and idempotency semantics MUST be explicit.

Standard resource semantics MUST NOT be distorted merely to avoid a custom action.

## 4. Request And Response Contracts

Public API models MUST be treated as boundary contracts. Persistence entities and framework-specific models MUST NOT be exposed directly unless that exposure is an intentional, verified contract decision.

Request models MUST expose only client-writable data. Server-owned, security-sensitive, derived, or internal fields MUST NOT become writable merely because they exist in an internal model.

Response models SHOULD expose only contractually meaningful information. Internal implementation state SHOULD NOT be exposed for convenience.

Field names and values MUST have stable business meaning. A field MUST NOT be reused later with incompatible semantics.

### Validation

Externally controlled input MUST be validated before trusted business processing.

Transport and format validation MUST NOT be treated as sufficient validation of business rules, authorization, ownership, or cross-field invariants.

Validation rules SHOULD have one clear owning layer. Equivalent business rules SHOULD NOT be independently reimplemented across layers merely because each layer can validate them.

### Partial Updates

**TRIGGER:** An API supports partial updates.

The contract MUST distinguish all materially different states that clients can express, including omitted, null, empty, cleared, and reset-to-default states when applicable.

A partial-update representation MUST NOT rely on accidental serializer behavior to define business semantics.

### Domain-Sensitive Values

Values whose interpretation depends on units or context MUST define those semantics explicitly.

Timestamps MUST define timezone or offset semantics when they represent an instant in time. Monetary values MUST define currency semantics when multiple currencies are possible. Quantities MUST define units when the unit is not inherent in the field contract.

A value MUST NOT be silently relabeled or interpreted in a different currency, unit, or timezone.

### Machine-Readable Schema

Externally consumed APIs SHOULD have machine-readable contract documentation when the project supports it. The schema SHOULD describe request fields, response fields, validation constraints, error responses, authentication requirements, and compatibility-relevant behavior.

A generated schema MUST NOT be treated as authoritative when it contradicts verified runtime behavior or an explicit contract. The inconsistency MUST instead be resolved.

## 5. Errors And Failure Semantics

HTTP status codes MUST represent the HTTP-level result. Application error codes MUST NOT replace correct HTTP status semantics.

Authentication failure, authorization failure, missing resources, conflicts, invalid input, rate limits, and unexpected server failures SHOULD use status semantics appropriate to the actual failure.

An API SHOULD use one consistent structured error model within a contract. The model SHOULD provide a machine-readable error type or code, a human-readable message, and field-level details when validation failures require them.

RFC 9457 Problem Details SHOULD be used when no established project error contract exists and it fits the API's requirements.

Error responses MUST NOT expose secrets, credentials, stack traces, SQL statements, internal filesystem paths, or unnecessary infrastructure details.

Failure responses MUST preserve enough stable information for clients to distinguish materially different recovery behavior. Internal exception class names MUST NOT be used as public error contracts merely because they are available.

## 6. Collections And Queries

Collection APIs MUST define behavior that remains safe as the collection grows.

**TRIGGER:** A collection is not reliably bounded to a small result set.

Pagination MUST be provided. The contract MUST define request parameters, maximum page size or equivalent limit, ordering behavior, continuation behavior, and empty-result behavior.

Cursor or token-based pagination SHOULD be preferred for large or frequently changing collections when offset traversal cannot provide stable or efficient behavior. Offset pagination MAY be used when its consistency and performance characteristics satisfy verified requirements.

### Filtering And Sorting

Query parameters MUST represent intentional API capabilities. Internal entity fields, SQL fragments, repository properties, or reflection-discovered fields MUST NOT automatically become externally selectable filters or sort keys.

Arbitrary query languages MUST NOT be exposed unless the language itself is an explicitly designed, validated, authorized, and bounded contract.

Supported filtering and sorting semantics SHOULD remain stable once consumers depend on them.

### Resource Limits

**TRIGGER:** A request or query can consume significant memory, CPU, storage, database work, bandwidth, or processing time.

The API MUST define enforceable limits appropriate to the verified workload. Relevant limits MAY include request size, page size, batch size, query complexity, processing duration, or uploaded content size.

Limits SHOULD fail explicitly rather than permitting uncontrolled resource consumption.

## 7. Operations And State

### Idempotency

HTTP method idempotency semantics MUST be preserved.

**TRIGGER:** A non-idempotent operation can be retried and duplicate execution can produce incorrect business effects.

The API MUST define an idempotency mechanism or another verified duplicate-prevention strategy. The mechanism MUST define uniqueness scope, replay behavior, expiration when applicable, and failure behavior.

Automatic retry MUST NOT be assumed safe solely because the transport can retry the request.

### Concurrency

**TRIGGER:** Concurrent updates can overwrite changes or violate a business invariant.

The API SHOULD expose or enforce explicit concurrency control appropriate to the requirement. Valid mechanisms MAY include entity versions, ETags, conditional requests, atomic operations, or database-enforced invariants.

Silent last-write-wins behavior SHOULD NOT be used when it violates a verified consistency requirement.

### Long-Running Operations

**TRIGGER:** An operation cannot reliably complete within the normal request lifecycle.

The API MUST NOT report synchronous completion before the requested business operation has actually completed.

An asynchronous contract SHOULD expose explicit operation identity and state. The contract MUST define completion detection and failure reporting, and SHOULD define cancellation when cancellation is a supported business capability.

### Batch Operations

**TRIGGER:** A verified client workflow repeatedly performs the same operation across multiple resources and per-resource requests create material cost or complexity.

A batch operation MAY be introduced. Its contract MUST define maximum batch size, atomicity, partial failure behavior, and ordering guarantees when ordering matters.

A batch API MUST NOT leave partial execution and retry semantics undefined.

## 8. Compatibility And Evolution

Compatibility MUST be evaluated against verified consumers and published contracts rather than hypothetical future clients.

Once consumers depend on behavior, changes to field presence, field meaning, requiredness, identifier semantics, status behavior, error semantics, ordering, pagination, authorization, or side effects MUST be evaluated for compatibility impact.

Compatible evolution SHOULD prefer additive changes such as new optional fields, new endpoints, or new optional request capabilities when those changes preserve existing semantics.

Versioning MUST NOT be introduced only because future breaking changes are theoretically possible.

**TRIGGER:** Multiple incompatible contracts must be supported simultaneously.

A versioning strategy MUST be defined. Whether the version is represented in a path, header, media type, or another mechanism is a project-level contract decision.

Deprecated behavior SHOULD have a defined removal or migration path when removal is intended. Compatibility branches MUST NOT remain indefinitely after their verified consumers and migration requirements no longer exist.

## 9. Security And Trust Boundaries

API boundaries MUST apply the relevant requirements from `security-guidelines.md` when they introduce or change security-sensitive behavior.

Protected operations MUST define authentication and authorization requirements. Authentication MUST NOT be treated as authorization.

**TRIGGER:** A request accesses or modifies data scoped to a user, tenant, organization, account, or other ownership boundary.

Authorization MUST be checked against the addressed resource and requested operation. Client-side hiding of an operation MUST NOT be treated as an authorization control.

Externally supplied identifiers, URLs, field selectors, filenames, redirects, filters, and other control inputs MUST remain untrusted until the applicable validation and authorization requirements are satisfied.

## 10. Operational Concerns

### Rate Limits And Quotas

**TRIGGER:** An operation is abuse-sensitive, expensive, externally exposed at meaningful scale, or consumes scarce shared resources.

The API SHOULD define an enforceable rate limit, quota, or equivalent resource control when existing infrastructure does not already provide sufficient protection.

Client-visible limiting behavior SHOULD define retry semantics and SHOULD use standard HTTP mechanisms or an established project contract. Clients SHOULD NOT be required to infer throttling from arbitrary server failures.

### Observability

APIs SHOULD support correlation between client-visible failures and server-side diagnostics when operational investigation requires it.

Request or trace identifiers MAY be exposed when useful, but they MUST NOT contain secrets or sensitive business information.

Observability data MUST NOT redefine the public business contract accidentally.

## 11. API Design Smells

API smells are investigation signals. A smell MUST NOT be treated as sufficient evidence that redesign is required.

| Smell | Signal |
|---|---|
| HTTP 200 for every outcome | HTTP failure semantics may be hidden inside the payload |
| Persistence entity exposed directly | Internal storage contract may have leaked into the API |
| GET changes business state | HTTP safety semantics are violated |
| Forced CRUD for domain commands | Domain behavior may be obscured to avoid a meaningful action |
| Generic action endpoints | Internal service methods may be exposed as public contract |
| PATCH without null and omission semantics | Partial-update behavior is ambiguous |
| Unbounded collection | Availability and latency may degrade with data growth |
| Arbitrary filter or sort fields | Internal query implementation may be exposed |
| Multiple identifiers without distinct meaning | Resource identity is ambiguous |
| Client controls server-owned fields | Trust boundary is violated |
| Async work returns completed success | Operation lifecycle is hidden |
| Batch API without partial-failure semantics | Retry and recovery behavior is undefined |
| Version added for ordinary compatible change | Compatibility strategy may be overdesigned |
| Duplicate error formats | Failure contract is inconsistent |
| Resource path mirrors implementation structure | API contract is coupled to internals |

**TRIGGER:** An API smell is encountered within the scope of the current change.

Its concrete contract, correctness, security, compatibility, or operational impact MUST be established before behavior or structure is changed because of the smell.
