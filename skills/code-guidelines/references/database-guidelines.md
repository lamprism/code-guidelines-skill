# Database And Persistence Guidelines

These requirements apply to persisted business data, database schemas, persistence models, migrations, queries, transactions, concurrency, and data lifecycle behavior. They use the normative model defined by `../SKILL.md`.

The goal is to design the smallest data model that correctly satisfies verified business, access, lifecycle, consistency, and operational requirements. Theoretical normalization, future-proofing, ORM convenience, and storage ideology MUST NOT replace evidence-driven modeling.

## 1. Persistence Boundaries

Persisted business data MUST have a clear ownership boundary.

Persistence models, repositories, DAOs, queries, and schema structures SHOULD remain within the component that owns the persisted concept. Cross-component code MUST NOT directly modify another component's owned persistence data as a shortcut around an explicit business or integration contract.

Cross-boundary access SHOULD use domain interfaces, application contracts, read models, APIs, events, or another explicit integration mechanism appropriate to the verified requirement.

Persistence implementation details MUST NOT become public or cross-domain contracts accidentally.

A shared database MUST NOT be treated as evidence that every component owns every table.

## 2. Minimal Viable Data Model

Database structures MUST be driven by verified present-day requirements.

A field, table, index, relationship, abstraction table, compatibility column, or duplicate representation MUST NOT be introduced only because it could theoretically be useful in the future.

Before introducing or decomposing persisted data, the design SHOULD establish the relevant:

- business owner;
- readers and writers;
- lifecycle;
- query patterns;
- filtering and sorting requirements;
- update behavior;
- consistency requirements;
- retention requirements.

A more normalized, decomposed, generic, or extensible model SHOULD NOT be preferred solely because it appears theoretically purer.

### Access-Pattern Driven Design

Known business operations and access patterns SHOULD guide storage design.

A schema MUST NOT be optimized for imaginary future queries. Performance-oriented structure MUST satisfy the performance decision gate defined by `../SKILL.md`.

### Meaningful Grouping

**TRIGGER:** A business concept contains multiple related attributes.

Attributes SHOULD remain grouped as one meaningful structure when they share ownership, lifecycle, validation, update behavior, and business meaning and do not require independent database behavior.

A structured concept MUST NOT be decomposed into independent columns or tables solely because each attribute can be represented separately.

Independent representation SHOULD be introduced when it provides a concrete requirement or benefit such as:

- filtering;
- sorting;
- indexing;
- database constraints;
- independent lifecycle;
- independent updates;
- independent authorization or consistency;
- reporting or aggregation requirements.

The decision to group data MUST NOT be interpreted as a requirement to use any specific physical storage format.

## 3. Schema Design

Schema structures MUST represent stable data semantics and ownership rather than temporary application structure.

Tables SHOULD have clear ownership, lifecycle, identity, and integrity rules. Tables MUST NOT be introduced merely to mirror temporary DTOs, UI forms, framework classes, or object hierarchies.

Database constraints SHOULD enforce invariants that are stable, local to the stored data, and unsafe to leave solely to application convention. Constraints MUST NOT duplicate volatile business workflows merely because the database can express them.

### Identifiers

Persisted entities MUST have stable identity semantics when identity is required.

Identifier strategy MUST be chosen from actual persistence, distribution, integration, and lifecycle requirements. A database-generated identifier MUST NOT automatically become an external API identifier.

Natural identifiers MAY be used when they represent stable business identity. Surrogate identifiers MAY be used when they better isolate persistence identity from mutable business attributes.

### Relationships

**TRIGGER:** A persisted relationship is introduced.

The relationship MUST have a verified ownership, integrity, lifecycle, or query requirement.

Foreign keys SHOULD be used when the database is responsible for enforcing a stable referential integrity rule and the participating data shares a compatible lifecycle and deployment boundary.

A relationship MUST NOT be introduced only to reproduce object navigation or because two concepts can theoretically reference each other.

Relationship design SHOULD establish ownership, deletion semantics, loading behavior, cardinality, and update behavior.

### Table And Column Naming

Database identifiers MUST communicate stable business or persistence meaning.

Naming conventions MUST be consistent within the same schema unless an existing external contract requires otherwise.

Table names SHOULD represent business or persistence concepts rather than technical wrappers. Prefixes or suffixes such as `tbl`, `table`, `entity`, `service`, or `data` SHOULD NOT be added when they do not add semantic information.

Singular and plural table naming MAY both be valid. A project SHOULD use one established convention consistently rather than changing names solely to satisfy a universal style preference.

Names SHOULD avoid unclear abbreviations. Established domain or industry abbreviations MAY be used when their meaning is unambiguous in the relevant context.

Column names MUST describe the meaning of stored data. Generic names such as `value`, `data`, `info`, `type`, `status`, or `flag` SHOULD NOT be used when the surrounding table context does not make their meaning unambiguous.

Primary keys MAY use `id` when table context makes the identity clear. Foreign-key columns SHOULD identify the referenced concept when omission would make the relationship ambiguous, such as `user_id`, `order_id`, or `payment_id`.

Redundant context SHOULD NOT be repeated in every column name when the table already provides that context and ambiguity does not exist.

### Temporal Columns

Columns representing instants in time SHOULD use a consistent semantic naming convention. The `_at` suffix is RECOMMENDED for time points such as `created_at`, `updated_at`, `expires_at`, `published_at`, and `deleted_at` when consistent with the project convention.

Columns representing calendar dates rather than instants SHOULD use a distinct convention. The `_date` suffix is RECOMMENDED for business dates such as `trade_date`, `report_date`, and `birth_date` when consistent with the project convention.

A timestamp and a business date MUST NOT be treated as interchangeable concepts.

### Boolean Columns

Boolean columns SHOULD express a clear true or false state. Names such as `is_active`, `has_access`, or `can_retry` MAY be used when they make the state unambiguous.

Generic names such as `flag` or `status` MUST NOT carry boolean semantics when the true and false meanings cannot be understood from the name and table context.

### Enumerated Values

Columns storing closed business states or types MUST have defined value semantics. Column names SHOULD identify the represented concept when a generic name such as `status` or `type` would be ambiguous within the table.

Persisted enum representation MUST remain stable or have an explicit migration strategy. Unknown persisted values MUST have a defined handling strategy and MUST NOT be silently discarded.

### Numeric Values And Units

Numeric columns MUST have explicit unit and precision semantics when interpretation is not inherent in the business concept.

A name SHOULD include the unit when omission would be ambiguous, such as `timeout_seconds`, `amount_cents`, or `retry_count`.

A numeric value MUST NOT be silently interpreted using a different unit, scale, currency, or precision.

## 4. Structured Data Representation

The physical storage representation SHOULD match verified access, lifecycle, consistency, validation, and evolution requirements.

A meaningful business structure MAY be represented as separate columns, an embedded structured value, a document or JSON value, a related table, or another storage model supported by the selected database.

No representation MUST be preferred solely because it is more relational, more normalized, more document-oriented, or more convenient for a framework.

### Embedded Structured Data

Structured embedded data MAY be used when the data shares ownership and lifecycle with its parent, has limited independent querying, and is naturally updated or validated as a unit.

Embedded structured data SHOULD NOT hide attributes that require frequent independent filtering, sorting, indexing, constraints, aggregation, or lifecycle management.

A generic document or JSON field MUST NOT become an unstructured dumping ground for unrelated data merely to avoid schema design.

Conversely, stable structured data MUST NOT be decomposed into many independent columns solely to avoid document or embedded storage.

Structured column names SHOULD describe the contained business concept, such as `notification_preferences` or `shipping_address`. Generic names such as `data`, `payload`, `extra`, or `misc` SHOULD NOT be used unless the data is intentionally generic by contract.

## 5. Persistence Models

Persistence models SHOULD represent persistence responsibilities and MAY differ from domain models, API models, and read models when those models have different responsibilities.

Persistence models MUST NOT be exposed across boundaries merely to avoid mapping code.

Persistence objects SHOULD NOT become containers for unrelated business workflows solely because they already contain the stored data.

Mutable collections or implementation-specific persistence state SHOULD NOT be exposed in ways that allow callers to bypass invariants.

### ORM Behavior

**TRIGGER:** An ORM or persistence framework provides implicit loading, cascading, lifecycle, dirty tracking, query generation, or transaction behavior that materially affects correctness or performance.

The relevant behavior MUST be verified before it is relied upon.

Fetching, cascading, ownership, and deletion behavior MUST be intentional. Eager loading SHOULD NOT be used as a blanket fix for query behavior. Lazy loading SHOULD NOT be allowed to leak across boundaries where the required persistence context is unavailable or where it creates hidden query behavior.

Bidirectional relationships SHOULD NOT be introduced by default. They MAY be used when both navigation directions represent verified behavior and their lifecycle and synchronization semantics are understood.

## 6. Schema Migration And Evolution

**TRIGGER:** A change modifies schema structure, constraints, indexes, persisted representation, or existing stored data.

The change MUST use the repository's established migration mechanism when one exists.

The migration MUST evaluate existing data, application compatibility, deployment order, partial execution, and recovery behavior.

Destructive schema changes MUST NOT be performed without a verified migration strategy. Destructive changes include dropping required data, incompatible type changes, narrowing valid values, changing identifier semantics, or introducing constraints that existing data may violate.

### Expand And Contract

**TRIGGER:** A schema and all of its consumers cannot be changed atomically and incompatible old and new representations would otherwise overlap during deployment.

An expand-and-contract migration SHOULD be used unless another verified migration strategy provides equivalent safety.

A typical sequence MAY introduce a compatible representation, deploy compatible readers or writers, migrate existing data, verify dependency removal, and then remove obsolete structure.

Compatibility columns, dual writes, and temporary migration branches MUST NOT remain after their verified migration purpose has ended.

Data backfills MUST define restart and failure behavior when partial execution is possible.

## 7. Query And Performance

Queries SHOULD be designed around verified access patterns.

A query MUST NOT depend on unspecified row ordering or other accidental database behavior when correctness depends on deterministic results.

Queries SHOULD retrieve only data required by the operation when unnecessary loading creates material cost or obscures intent.

**TRIGGER:** A query can return an unbounded or materially large result set.

The query MUST define a bound, pagination strategy, streaming strategy, or another verified resource-control mechanism.

### Indexes

Indexes SHOULD be introduced for verified query, uniqueness, ordering, or performance requirements.

An index MUST NOT be added solely because a column appears in a query or might be queried in the future. Index design SHOULD account for write cost, storage cost, selectivity, ordering, and actual query shape.

Performance claims about an index SHOULD be verified through query plans, benchmarks, production evidence, expected scale, or equivalent database evidence when the decision is material.

### Query Amplification

**TRIGGER:** ORM navigation, repeated repository calls, or collection traversal may issue additional database queries.

Actual query behavior MUST be inspected when query amplification can materially affect the expected workload.

N+1 queries, repeated identical queries, unnecessary joins, and equivalent amplification SHOULD be eliminated when they create a verified or expected material cost.

## 8. Transactions And Consistency

Transactions MUST represent meaningful consistency boundaries.

Operations that must succeed or fail together SHOULD share an appropriate transaction boundary. Unrelated workflows SHOULD NOT be placed in one transaction merely to make the implementation appear atomic.

Transaction scope SHOULD be kept as small as correctness permits. External network calls SHOULD NOT be performed inside a database transaction unless the consistency model explicitly requires the interaction and its failure behavior is understood.

Isolation level MUST satisfy the verified consistency requirement. Stronger isolation MUST NOT be selected automatically without considering contention, latency, and throughput impact.

Database transactions MUST NOT be assumed to provide atomicity across external systems.

## 9. Concurrency Control

**TRIGGER:** Concurrent writes can cause lost updates, duplicate effects, invalid state transitions, or violated invariants.

The system MUST define a concurrency strategy appropriate to the invariant.

Valid mechanisms MAY include optimistic locking, version fields, uniqueness constraints, atomic conditional updates, explicit database locking, or serialization through another verified mechanism.

Database locks SHOULD NOT be used as the default solution for cross-system coordination. Distributed coordination requirements MUST be evaluated at the system boundary rather than inferred from local database locking alone.

Concurrency protection MUST be verified against the actual race it is intended to prevent.

## 10. Data Lifecycle

Persisted business data MUST have defined ownership and lifecycle behavior when retention or deletion matters to correctness, privacy, compliance, or operations.

Relevant lifecycle behavior MAY include creation, update, archival, retention, deletion, restoration, and purge.

### Soft Delete

Soft deletion MAY be used when historical visibility, retention, restoration, audit, or business requirements justify it.

Soft deletion MUST NOT be introduced solely because physical deletion feels unsafe.

**TRIGGER:** Soft deletion is used.

Normal query behavior, uniqueness behavior, relationships, restoration semantics, and eventual purge behavior MUST be defined where applicable.

A soft-deleted record MUST NOT silently participate in normal business behavior when the contract treats it as deleted.

## 11. Special Data Representation

### Time

Persisted time values MUST have explicit semantics.

Instants MUST NOT depend on an implicit server-local timezone. Business dates, local wall-clock times, and absolute instants MUST remain distinct when they represent different concepts.

The chosen storage representation MAY vary by database and project as long as the required precision, timezone semantics, ordering, and interoperability are preserved.

### Money

Monetary values MUST preserve required precision and currency semantics.

Floating-point representation MUST NOT be used for monetary values when binary floating-point rounding can violate the required financial precision.

An amount MUST NOT be stored or interpreted without currency semantics when multiple currencies are possible. Native amounts MUST NOT be relabeled as another currency without an explicit conversion rule.

## 12. Database Design Smells

Database smells are investigation signals. A smell MUST NOT be treated as sufficient evidence that redesign is required.

| Smell | Signal |
|---|---|
| Premature decomposition | Data was split without query, lifecycle, integrity, or update benefit |
| Column explosion | One business concept is scattered across many independently managed fields |
| Over-normalization | Structural complexity exists without a verified business or integrity benefit |
| Generic document dumping | Unrelated structured data is hidden in opaque storage without clear semantics |
| Stable structure decomposed mechanically | Columns or tables exist only because attributes can be represented separately |
| Future-proof schema | Fields, tables, indexes, or compatibility paths exist only for hypothetical needs |
| Cross-boundary persistence access | Data ownership is bypassed through shared storage |
| Persistence model exposed externally | Storage representation has become an accidental contract |
| Uncontrolled ORM behavior | Loading, cascading, lifecycle, or generated queries are not understood |
| Bidirectional relationship by default | Navigation structure may be driving persistence design |
| Missing migration strategy | Existing data or deployment compatibility is at risk |
| Query without access-pattern evidence | Schema or query shape may be optimized for imagined usage |
| Unbounded query | Resource use grows with data size without a control mechanism |
| Index without justification | Write and storage cost was added without verified benefit |
| Large transaction scope | Consistency boundaries and operational work may be mixed |
| Soft delete without lifecycle policy | Deleted data semantics are ambiguous |
| Ambiguous identifier or unit | Stored values cannot be interpreted safely from their contract |
| Inconsistent naming | Schema meaning depends on local tribal knowledge rather than stable convention |

**TRIGGER:** A database smell is encountered within the scope of the current change.

Its concrete correctness, integrity, ownership, migration, performance, or maintenance impact MUST be established before schema or persistence structure is changed because of the smell.
