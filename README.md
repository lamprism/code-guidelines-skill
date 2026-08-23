# Code Guidelines Skill

A reusable coding-agent skill for evidence-driven engineering, focused changes, Clean Code, YAGNI, security-aware development, HTTP API contracts, database and persistence design, testing discipline, Java/JVM conventions, and repository-specific guideline discovery.

## Structure

```text
skills/
└── code-guidelines/
    ├── SKILL.md
    └── references/
        ├── engineering-guidelines.md
        ├── security-guidelines.md
        ├── rest-api-guidelines.md
        ├── database-guidelines.md
        ├── java-guidelines.md
        └── project-guidelines.md
```

## Usage

Load `skills/code-guidelines/SKILL.md` for code analysis, implementation, bug investigation, code review, refactoring, testing, security-sensitive engineering work, HTTP API design, and database or persistence changes.

The skill defines a shared normative model based on BCP 14 terminology and extends it with `TRIGGER` and `EXCEPTION` for conditional engineering rules. It routes to:

- `engineering-guidelines.md` for language-independent engineering rules;
- `security-guidelines.md` for trust boundaries and security-sensitive behavior;
- `rest-api-guidelines.md` for HTTP API contracts, resources, methods, errors, collections, compatibility, and API evolution;
- `database-guidelines.md` for persistence boundaries, data modeling, schemas, migrations, queries, transactions, concurrency, and data lifecycle;
- `java-guidelines.md` for Java, Kotlin, and JVM-specific rules;
- `project-guidelines.md` for discovering and maintaining repository-specific conventions.

Repository-specific rules should remain concise deltas over the general, security, API, database, and language guidelines rather than duplicating them.
