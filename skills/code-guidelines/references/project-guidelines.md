# Project Guidelines

Project-level guidelines define stable repository-specific decisions that cannot be derived from the general, security, API, or language guidelines. They use the normative model defined by `../SKILL.md`.

Project guidelines MUST remain repository-specific deltas rather than duplicate coding handbooks.

## 1. What Belongs In Project Guidelines

A project-level rule MUST record a repository-specific decision that contributors need in order to make consistent changes.

Typical project-specific areas include:

| Area | Examples |
|---|---|
| Architecture | Module ownership, allowed dependency directions, adapter boundaries, shared-kernel rules |
| Domain | Canonical identifiers, domain terminology, value representations, project-specific invariants |
| Persistence | Persistence package boundaries, ID strategies, schema naming, auditing, repository patterns, transaction conventions |
| Database migration | Migration immutability, schema-change workflow, data-migration policy, rollback expectations |
| API | Base paths, resource naming, DTO conventions, versioning, compatibility policy, event or message formats |
| Framework | Project-specific Spring conventions, bean contracts, extension mechanisms, annotation conventions |
| JVM | Project-specific Java or Kotlin source layout, interoperability rules, annotation requirements |
| Build | Required build commands, generated-source rules, dependency management, code-generation workflows |
| Testing | Test layout, integration-test infrastructure, fixtures, test commands, architecture-test rules |
| Frontend | Component architecture, shared layout ownership, styling conventions, state-management boundaries |
| Exceptions | Explicit repository-specific exceptions to a general, security, API, or language guideline |

Categories that do not exist in the repository MUST NOT be added merely for completeness.

General rules such as meaningful naming, SOLID, exception handling, testing requirements, composition over inheritance, input validation, and verification SHOULD NOT be duplicated in project-level guidance.

Language-wide rules such as `java.time`, nullability, resource management, or general JSON practices SHOULD NOT be repeated unless the repository intentionally specializes or overrides them.

### Project Rules Are Deltas

A project-level rule SHOULD answer a question the shared guidelines cannot answer, for example:

- Which module owns a domain concept?
- Which identifier is canonical in this repository?
- Which package owns persistence for a feature?
- Which database timestamp representation does this project use?
- Are applied migrations immutable?
- Which Spring bean contract convention is established here?
- Where does this repository place Kotlin source files?
- Which integration-test infrastructure is required?
- Which generated files are committed and how are they regenerated?

A project-level rule SHOULD NOT merely restate that code must be readable, tested, secure, or maintainable.

## 2. Project Rules Require Evidence

A repository pattern MUST NOT be promoted to a project-wide rule without supporting evidence.

Evidence SHOULD be evaluated in approximately this order of authority:

1. Explicit project or team requirements.
2. Existing repository architecture or engineering documentation.
3. Build, formatter, linter, compiler, or static-analysis configuration.
4. Framework configuration.
5. Schema, migrations, public API contracts, or protocol definitions.
6. Architecture tests or other executable constraints.
7. Multiple consistent and current implementations across the relevant codebase.

A single implementation MAY be evidence of a local pattern, but MUST NOT by itself establish a repository-wide rule.

Existing code MUST NOT automatically be treated as authoritative because it MAY represent legacy behavior, a local exception, an incomplete migration, code scheduled for replacement, or an incidental implementation choice.

**TRIGGER:** Evidence is insufficient to establish that a pattern is mandatory.

- The pattern MUST NOT be documented with **MUST** or **MUST NOT**.
- It MAY be documented as an observed pattern when doing so is useful and clearly distinguished from a normative requirement.

### Tooling Is Evidence

Repository tooling SHOULD be treated as strong evidence of intended mechanical conventions.

Applicable formatter configuration, linter configuration, compiler options, static-analysis rules, architecture tests, dependency rules, code-generation configuration, build scripts, schema tooling, and test configuration SHOULD be inspected before documenting equivalent prose rules.

A purely mechanical rule already enforced completely and unambiguously by tooling SHOULD NOT be duplicated in prose unless additional semantic context is required.

## 3. Normative Project Rules

Project-level rules MUST use the normative language defined by `../SKILL.md`.

- **MUST** and **MUST NOT** MUST be used only for verified mandatory requirements.
- **SHOULD** and **SHOULD NOT** SHOULD be used for established defaults that permit evidence-supported deviation.
- **MAY** MUST be used only when a behavior is explicitly permitted but not required.
- Conditional project rules SHOULD use **TRIGGER** when applicability cannot be stated unambiguously in the requirement itself.
- Known exceptions to an otherwise applicable rule MUST be expressed explicitly and narrowly through **EXCEPTION**.
- Lowercase words such as `must`, `should`, `may`, `prefer`, `normally`, `usually`, `when appropriate`, or `consider` SHOULD NOT be used to express normative force when an applicable BCP 14 keyword can state the intended strength precisely.

**TRIGGER:** A project rule uses **SHOULD** or **SHOULD NOT**.

- The deviation conditions SHOULD be inferable from the rule's rationale, stated tradeoff, or explicit **EXCEPTION**.
- Preference or convenience alone MUST NOT justify deviation from the established default.

## 4. Establishing And Maintaining Project Guidelines

Before establishing or changing project-level guidelines, the relevant repository evidence MUST be inspected. The investigation SHOULD include, when applicable:

1. Existing engineering and architecture documentation.
2. Build and static-analysis configuration.
3. Module, package, and dependency boundaries.
4. Several representative current implementations.
5. Relevant tests and architecture tests.
6. Framework, persistence, API, and schema configuration.
7. Conflicting patterns and their historical or architectural context.

Repository-specific decisions MUST be separated from general engineering, security, API, and language rules.

Only rules that future contributors need before making related changes SHOULD be recorded.

When documenting a rule:

- Mandatory wording MUST be supported by evidence.
- Meaningful exceptions MUST be explicit and narrowly scoped.
- An arbitrary-looking rule SHOULD include enough rationale to preserve its engineering or domain intent.
- Speculative future architecture, technologies, compatibility requirements, or extension points MUST NOT be documented as current project requirements.
- Historical rules MUST NOT remain beside their replacements as parallel active guidance.
- Obsolete project rules MUST be updated or removed.

### Conflicting Existing Patterns

**TRIGGER:** Existing code contains materially conflicting patterns for the same engineering decision.

The investigation MUST determine, when evidence exists:

1. whether one pattern intentionally replaces another;
2. whether patterns belong to different modules, layers, domains, or architectural contexts;
3. whether an actual compatibility constraint explains the difference;
4. whether one pattern is legacy code or part of an incomplete migration.

A unified convention MUST NOT be invented merely to remove visible inconsistency.

Unrelated existing code MUST NOT be migrated while establishing a convention for a focused change.

If evidence does not establish a single intended rule, the ambiguity MUST remain explicit rather than being converted into an unsupported mandatory convention.

### Database Migration Policy

Database migrations MUST NOT be assumed immutable or editable without repository evidence.

**TRIGGER:** Database changes or migration rules are relevant to a task.

The project's actual policy MUST be established from migration tooling, documentation, deployment practices, and existing history before migration history is changed.

A project guideline MAY define:

- whether applied migrations are immutable;
- how corrective migrations are created;
- how destructive schema changes are handled;
- how data migrations are separated from schema migrations;
- whether rollback migrations are required;
- how compatibility is maintained during rolling deployments.

Applied migration history MUST NOT be rewritten unless verified project policy explicitly permits it.

### Generated And Tool-Managed Files

**TRIGGER:** A repository contains generated or tool-managed files relevant to normal development.

Project guidelines SHOULD identify, when not already self-evident from tooling:

- whether generated files are committed;
- which source definitions own them;
- which command regenerates them;
- whether lockfiles are committed;
- which tool owns lockfile updates;
- whether generated API clients or schemas may be edited manually.

A committed file MUST NOT be assumed manually editable merely because it exists in version control.

## 5. Admission Checklist

Before adding a project-level rule, all applicable admission conditions MUST be satisfied:

| Question | Required result |
|---|---|
| Is the rule specific to this repository? | Yes |
| Is it supported by concrete evidence? | Yes |
| Is it a stable engineering decision rather than an incidental implementation detail? | Yes |
| Is it not already fully covered by a shared guideline? | Yes |
| Does it provide information tooling alone cannot fully communicate? | Usually yes |
| Would contributors need it before making related changes? | Yes |
| Is mandatory wording supported by evidence? | Yes, for **MUST** or **MUST NOT** |
| Are known exceptions explicitly bounded? | Yes, when exceptions exist |

If the admission conditions are not satisfied, the rule SHOULD NOT be added as project-level normative guidance.

A project-specific guideline document MAY use the following structure when applicable:

    # Project Coding Guidelines

    ## Architecture
    ## Domain Conventions
    ## Persistence
    ## API And Protocols
    ## Java And Framework Conventions
    ## Build And Tooling
    ## Testing
    ## Frontend
    ## Project-Specific Exceptions

Empty sections for technologies, layers, or architectural areas the repository does not use SHOULD NOT be created.
