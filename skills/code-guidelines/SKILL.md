# Code Guidelines

Use this skill for code analysis, feature implementation, bug investigation, code review, refactoring, and testing.

## 1. Rules And Workflow

When rules conflict, apply them in this order:

1. Agent operation and safety constraints.
2. Explicit user and task requirements.
3. Verified repository-specific rules.
4. Applicable language and technology guidelines.
5. General engineering guidelines.

Apply `references/engineering-guidelines.md` to all coding tasks.

Apply `references/java-guidelines.md` when Java, Kotlin, or related JVM technologies are involved.

Apply `references/project-guidelines.md` when discovering, establishing, evaluating, or modifying repository-specific engineering rules.

Interpret normative wording consistently:

| Wording | Meaning |
|---|---|
| `must` / `must not` | Hard requirement. Override only when a higher-priority requirement or an explicitly documented exception applies. |
| `prefer` / `by default` | Default choice. A concrete project constraint or verified reason may justify another choice. |
| `consider` | Investigation signal. Exercise engineering judgment; the wording does not itself require a code change. |

Existing code is evidence of project conventions, but a single implementation does not automatically establish a repository-wide rule.

Before modifying code:

1. Understand the requested behavior and affected scope.
2. Inspect repository instructions and relevant configuration.
3. Read the affected implementation, call path, and actual type or API definitions.
4. Inspect analogous implementations already present in the repository.
5. Determine the solution from concrete evidence.
6. Choose the smallest change that correctly satisfies the requirement.

If multiple materially different interpretations remain and the choice affects destructive operations, money, permissions, security, persistent data, or public contracts, clarify the requirement before implementation.

Do not introduce interfaces, strategies, configuration options, compatibility branches, fallbacks, defensive behavior, public APIs, or dependencies for hypothetical future requirements.

## 2. Task-Specific Requirements

### Bug Investigation

1. Establish the observed incorrect behavior.
2. Reproduce the problem when practical.
3. Trace the actual execution path.
4. Gather evidence from code, definitions, logs, tests, runtime results, or persisted data.
5. Form a falsifiable root-cause hypothesis.
6. Verify the hypothesis before declaring it the root cause.
7. Determine which component violates the intended contract before changing either caller or callee.
8. Implement the smallest fix that addresses the verified cause.
9. Add regression coverage when practical.
10. Verify the reported scenario after the fix.

Do not treat suspicious-looking code, correlation, or an untested hypothesis as a confirmed root cause.

Clearly distinguish observations, hypotheses, and verified conclusions.

Do not broaden a callee's contract, add null handling, introduce fallback behavior, or weaken validation merely because one caller currently violates the established contract.

### Feature Implementation

1. Identify the existing domain concepts and the components that own the relevant responsibilities.
2. Inspect the closest existing implementation before designing a new one.
3. Reuse existing abstractions when they correctly represent the required behavior.
4. Introduce a new abstraction only when the requirement creates a real responsibility, boundary, or variation point.
5. Keep new public API surface to the minimum required by actual consumers.
6. Implement the smallest complete behavior.
7. Add tests for new business behavior.

Do not design an extension system for hypothetical future implementations.

Do not introduce a new third-party dependency when the standard library or an existing project dependency already solves the problem clearly and adequately.

### Code Review

Prioritize findings in this order:

1. Correctness and data integrity.
2. Security.
3. Concurrency and resource safety.
4. Contract and compatibility correctness.
5. Missing or insufficient tests and verification.
6. Responsibility, dependency boundaries, and maintainability.
7. Performance problems supported by evidence.

Every substantive finding must identify concrete evidence and explain the actual or likely impact.

Code smells are investigation signals, not automatic defects.

Do not recommend refactoring solely to satisfy a pattern, stylistic preference, theoretical design principle, or arbitrary metric.

### Refactoring

- Refactoring must have a concrete motivation, such as duplicated business rules, misplaced responsibility, excessive coupling, or demonstrated cognitive complexity.
- Preserve externally observable behavior unless a behavior change is explicitly requested.
- Replace the existing implementation instead of introducing parallel variants such as `XxxV2`, `newXxx()`, or `XxxImpl2`.
- Do not expand a focused task into unrelated architectural cleanup.
- Establish relevant behavior with tests before and after significant refactoring when practical.
- Do not weaken an existing contract merely to simplify an implementation.

## 3. Verification And Delivery

After modifying code:

1. Run the most relevant focused tests.
2. Run broader regression or build checks when warranted by the affected scope.
3. Investigate failures instead of assuming they are unrelated.
4. Determine whether a failing test exposes a production defect, an incorrect test, or an intentionally changed requirement before modifying the test.
5. Inspect the final diff for accidental or unrelated changes.
6. Remove temporary debugging and intermediate artifacts.
7. Confirm that the implementation still follows applicable project conventions.

Do not make an incorrect implementation pass by deleting tests, skipping tests, weakening assertions, changing expected behavior without evidence, or introducing meaningless mocks.

If a test itself is incorrect, change it only after establishing the intended behavior from the actual contract or requirement.

If any verification cannot be completed, explicitly state what was verified, what was not verified, and why.

The final report must briefly state:

- what changed;
- why it changed;
- the affected scope;
- what was verified;
- relevant limitations, uncovered edge cases, or unverified behavior.

Do not claim successful completion beyond the available evidence.
