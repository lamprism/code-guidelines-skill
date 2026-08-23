# Project Guidelines

Project-level guidelines define stable repository-specific decisions that cannot be derived from the general engineering or language guidelines.

They describe the repository-specific delta. They are not a duplicate coding handbook.

## 1. What Belongs In Project Guidelines

Project guidelines should record decisions that contributors must know to make changes consistent with the repository.

Typical project-specific rules include:

| Area | Examples |
|---|---|
| Architecture | Module ownership, allowed dependency directions, adapter boundaries, shared-kernel rules. |
| Domain | Canonical identifiers, domain terminology, value representations, project-specific invariants. |
| Persistence | Persistence package boundaries, ID strategies, schema naming, auditing, repository patterns, transaction conventions. |
| Database migration | Migration immutability, schema-change workflow, data-migration policy, rollback expectations. |
| API | Base paths, resource naming, DTO conventions, versioning, compatibility policy, event or message formats. |
| Framework | Project-specific Spring conventions, bean contracts, framework extension mechanisms, annotation conventions. |
| JVM | Project-specific Java or Kotlin source layout, interoperability rules, annotation requirements. |
| Build | Required build commands, generated-source rules, dependency-management conventions, code-generation workflows. |
| Testing | Test layout, integration-test infrastructure, fixtures, test commands, architecture-test rules. |
| Frontend | Component architecture, shared layout ownership, styling conventions, state-management boundaries. |
| Exceptions | Explicit repository-specific exceptions to the general or language guidelines. |

Only include categories that actually exist in the repository.

Do not repeat general rules such as meaningful naming, SOLID, exception handling, testing requirements, composition over inheritance, input validation, or change verification.

Do not repeat language rules such as general `java.time`, nullability, resource-management, or JSON practices unless the repository intentionally specializes or overrides them.

### Project Rules Are Deltas

A project rule should answer a question that the general guidelines cannot answer.

For example:

- Which module owns a particular domain concept?
- Which identifier is canonical in this repository?
- Which package owns persistence for a feature?
- Which database timestamp representation does this project use?
- Does this repository use immutable migrations?
- Which Spring bean contract convention is established here?
- Where does this repository place Kotlin source files?
- Which test infrastructure is required for integration tests?
- Which generated files are committed, and how are they regenerated?

A project rule should not merely restate that code must be readable, tested, secure, or maintainable.

## 2. Project Rules Require Evidence

A repository pattern must not be promoted to a project rule without evidence.

Use evidence in roughly this order of authority:

1. Explicit user or team requirements.
2. Existing repository instruction or architecture documents.
3. Build, formatter, linter, compiler, or static-analysis configuration.
4. Framework configuration.
5. Schema, migrations, public API contracts, or protocol definitions.
6. Architecture tests or other executable constraints.
7. Multiple consistent and current implementations across the relevant codebase.

A single implementation is evidence of a pattern, but it is not sufficient by itself to establish a repository-wide rule.

Existing code is not automatically authoritative. It may be:

- legacy code;
- an intentional local exception;
- an incomplete migration;
- code scheduled for replacement;
- an implementation choice that was never intended as a convention.

When evidence is insufficient, describe the pattern as observed rather than mandatory.

Do not use `must` or `must not` when the evidence supports only a common convention.

### Tooling Is Evidence

Treat repository tooling as strong evidence of intended conventions.

Inspect applicable:

- formatter configuration;
- linter configuration;
- compiler options;
- static-analysis rules;
- architecture tests;
- dependency rules;
- code-generation configuration;
- build scripts;
- schema tooling;
- test configuration.

Do not duplicate purely mechanical tooling rules in prose unless contributors need additional semantic context that the tool itself cannot communicate.

## 3. Establishing And Maintaining Project Guidelines

Before establishing or changing project guidelines:

1. Inspect existing repository instructions and architecture documentation.
2. Inspect build and static-analysis configuration.
3. Identify module, package, and dependency boundaries.
4. Inspect several representative current implementations.
5. Inspect relevant tests and architecture tests.
6. Inspect framework, persistence, API, and schema configuration when applicable.
7. Identify conflicting patterns and determine whether they are intentional exceptions, different architectural contexts, migrations, or legacy code.
8. Separate repository-specific decisions from general engineering and language rules.
9. Record only rules that future contributors need to know before making related changes.

When documenting a rule:

- Use `must` or `must not` only for verified mandatory requirements.
- Use `prefer`, `by default`, or equivalent wording for established defaults that allow justified exceptions.
- State meaningful exceptions explicitly and narrowly.
- Explain the underlying engineering or domain reason when the rule would otherwise appear arbitrary.
- Do not add speculative future architecture, technologies, compatibility requirements, or extension points.
- Do not document behavior already enforced completely and unambiguously by tooling unless additional semantic context is necessary.
- Do not preserve historical rules alongside replacements.
- Update or remove obsolete rules instead of accumulating compatibility commentary.
- Do not create or modify a project guideline document unless the task explicitly requires documentation.

### Conflicting Existing Patterns

When existing code contains conflicting patterns:

1. Determine whether one pattern intentionally replaces another.
2. Check architecture documentation, migrations, tests, configuration, and version history when relevant.
3. Determine whether the patterns belong to different modules, layers, domains, or architectural contexts.
4. Identify actual compatibility constraints.
5. Determine whether one pattern is legacy code or an incomplete migration.
6. Ask the user when the intended direction remains materially ambiguous.

Do not invent a unified convention merely to remove inconsistency.

Do not migrate unrelated existing code while implementing a focused task.

### Database Migration Policy

Do not assume that database migrations are immutable or editable.

When database changes are relevant, determine the repository's actual policy from migration tooling, documentation, deployment practices, and existing history.

A project guideline may define:

- whether applied migrations must remain immutable;
- how corrective migrations are created;
- how destructive schema changes are handled;
- how data migrations are separated from schema migrations;
- whether rollback migrations are required;
- how compatibility is maintained during rolling deployments.

Do not rewrite migration history unless the verified project policy explicitly permits it.

### Generated And Tool-Managed Files

Identify which files are generated or owned by tools.

Project guidelines may define:

- whether generated files are committed;
- which source definitions own them;
- which command regenerates them;
- whether lockfiles are committed;
- which tool owns lockfile updates;
- whether generated API clients or schemas may be edited manually.

Do not infer that every committed file is intended for manual editing.

## 4. Admission Checklist

Before adding a rule to project-level guidelines, verify:

| Question | Required Result |
|---|---|
| Is the rule specific to this repository? | Yes |
| Is it supported by concrete evidence? | Yes |
| Is it a stable engineering decision rather than an incidental implementation detail? | Yes |
| Is it not already covered by general or language guidelines? | Yes |
| Does it provide information that tooling alone cannot fully communicate? | Usually yes |
| Would future contributors need to know it before making related changes? | Yes |
| Is mandatory wording supported by the evidence? | Yes, if `must` or `must not` is used |
| Are any exceptions known and explicitly bounded? | Yes, when exceptions exist |

If these conditions are not satisfied, the rule normally does not belong in project-level guidelines.

A project-specific guideline document may use the following structure when applicable:

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

Do not create empty sections for technologies, layers, or architectural areas the repository does not use.
