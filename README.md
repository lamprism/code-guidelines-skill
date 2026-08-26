# Code Guidelines Skill

Evidence-driven, simplicity-first engineering guidance for coding agents.

Code Guidelines gives agents a repeatable way to inspect a repository, apply only the guidance relevant to the task, make the smallest correct change, and verify the result. It covers general engineering practice plus API, persistence, security, concurrency, testing, observability, and Java/Kotlin/JVM concerns.

## Installation

This repository provides an agent skill, not a standalone application or library. Install it with:

```bash
npx skills add https://github.com/lamprism/code-guidelines-skill --skill code-guidelines
```

## Usage

After installation, `code-guidelines` is available to your coding agent. It applies the general engineering guide to every coding task and loads specialized guides when their documented triggers match the task. The guides are cumulative: a specialized guide refines the general guidance instead of replacing another applicable guide.

## Reference guides

Load only the guides relevant to the task.

| Guide | Use it for |
|---|---|
| [`engineering-guidelines.md`](skills/code-guidelines/references/engineering-guidelines.md) | General engineering, evidence, scope, design, implementation, review, and verification |
| [`security-guidelines.md`](skills/code-guidelines/references/security-guidelines.md) | Trust boundaries, credentials, untrusted input, files, network access, dependencies, and deployment security |
| [`rest-api-guidelines.md`](skills/code-guidelines/references/rest-api-guidelines.md) | HTTP API contracts, resources, methods, models, errors, compatibility, and evolution |
| [`database-guidelines.md`](skills/code-guidelines/references/database-guidelines.md) | Data models, schemas, migrations, queries, transactions, and data lifecycle |
| [`concurrency-guidelines.md`](skills/code-guidelines/references/concurrency-guidelines.md) | Threads, pools, locks, asynchronous work, coroutines, scheduling, races, and liveness |
| [`testing-guidelines.md`](skills/code-guidelines/references/testing-guidelines.md) | Test strategy, fixtures, isolation, determinism, failure paths, and flaky tests |
| [`java-guidelines.md`](skills/code-guidelines/references/java-guidelines.md) | Java, Kotlin, JVM APIs, interoperability, compiler/runtime behavior, and JVM-specific testing or build concerns |
| [`observability-guidelines.md`](skills/code-guidelines/references/observability-guidelines.md) | Logs, metrics, traces, health, lifecycle, runtime configuration, flags, and alerts |
| [`project-guidelines.md`](skills/code-guidelines/references/project-guidelines.md) | Repository conventions, architecture, tooling, generated files, and project-specific exceptions |

## Guidance model

- Shared guides define language-neutral concepts, contracts, and review signals.
- Language and runtime guides define syntax, APIs, interoperability, and runtime-specific behavior.
- Project guidance records repository-specific conventions and decisions; it refines shared and language rules without replacing them.
- Each substantive requirement should have one owning guide; other guides may add a trigger, boundary-specific consequence, or language/runtime mapping.

## Normative language

The uppercase keywords `MUST`, `MUST NOT`, `REQUIRED`, `SHOULD`, `SHOULD NOT`, `RECOMMENDED`, `MAY`, and `OPTIONAL` follow BCP 14 as defined by [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174). They are normative only when written in uppercase.

`TRIGGER` identifies the observable condition that makes a conditional rule applicable. `EXCEPTION` identifies an explicit, narrowly scoped condition that relaxes or disables an otherwise applicable rule. Missing information establishes neither a trigger nor an exception. See [`SKILL.md`](skills/code-guidelines/SKILL.md) for the complete model.

## Extending the skill

Keep shared guidance broadly applicable, evidence-backed, and triggerable. Put repository-specific decisions in `project-guidelines.md`. Add a new guide only when a domain has enough independent guidance to benefit from progressive disclosure, and keep [`SKILL.md`](skills/code-guidelines/SKILL.md) focused on the shared model, routing, workflow, and verification expectations.

## Scope and limitations

This skill supplements, rather than replaces, product requirements, repository rules, framework contracts, security review, operational limits, and language or toolchain documentation. Following the guidance or passing tests is not proof of complete correctness, security, compatibility, or performance.

## License

Released under the [MIT License](LICENSE).
