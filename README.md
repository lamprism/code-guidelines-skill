# Code Guidelines Skill

A reusable coding-agent skill for evidence-driven engineering, focused changes, Clean Code, YAGNI, security-aware development, testing discipline, Java/JVM conventions, and repository-specific guideline discovery.

## Structure

```text
skills/
└── code-guidelines/
    ├── SKILL.md
    └── references/
        ├── engineering-guidelines.md
        ├── security-guidelines.md
        ├── java-guidelines.md
        └── project-guidelines.md
```

## Usage

Load `skills/code-guidelines/SKILL.md` for code analysis, implementation, bug investigation, code review, refactoring, testing, and security-sensitive engineering work.

The skill defines a shared normative model based on BCP 14 terminology and extends it with `TRIGGER` and `EXCEPTION` for conditional engineering rules. It routes to:

- `engineering-guidelines.md` for language-independent engineering rules;
- `security-guidelines.md` for trust boundaries and security-sensitive behavior;
- `java-guidelines.md` for Java, Kotlin, and JVM-specific rules;
- `project-guidelines.md` for discovering and maintaining repository-specific conventions.

Repository-specific rules should remain concise deltas over the general, security, and language guidelines rather than duplicating them.
