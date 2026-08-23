# Code Guidelines Skill

A reusable coding-agent skill for evidence-driven engineering, focused changes, Clean Code, YAGNI, testing discipline, Java/JVM conventions, and repository-specific guideline discovery.

## Structure

```text
skills/
└── code-guidelines/
    ├── SKILL.md
    └── references/
        ├── engineering-guidelines.md
        ├── java-guidelines.md
        └── project-guidelines.md
```

## Usage

Load `skills/code-guidelines/SKILL.md` for code analysis, implementation, bug investigation, code review, refactoring, and testing.

The skill routes to:

- `engineering-guidelines.md` for language-independent engineering rules;
- `java-guidelines.md` for Java, Kotlin, and JVM-specific rules;
- `project-guidelines.md` for discovering and maintaining repository-specific conventions.

Repository-specific rules should remain concise deltas over the general and language guidelines rather than duplicating them.
