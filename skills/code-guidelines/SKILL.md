# Code Guidelines

Use this skill for code analysis, feature implementation, bug investigation, code review, refactoring, testing, and security-sensitive engineering work.

The goal is to produce the smallest correct change supported by evidence and consistent with the repository's verified contracts, architecture, and engineering requirements.

## 1. Normative Model

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this skill and its reference guidelines are to be interpreted as described in BCP 14, RFC 2119 and RFC 8174, when and only when they appear in all capitals.

| Term | Meaning |
|---|---|
| **MUST** / **REQUIRED** | Mandatory whenever the rule applies. |
| **MUST NOT** | Prohibited whenever the rule applies. |
| **SHOULD** / **RECOMMENDED** | Default engineering decision. Deviation requires a concrete, evidence-supported reason. |
| **SHOULD NOT** | Avoid by default. Deviation requires a concrete, evidence-supported reason. |
| **MAY** / **OPTIONAL** | Permitted but not required. |
| **TRIGGER** | Observable condition that makes a conditional rule applicable. |
| **EXCEPTION** | Explicit, narrowly scoped condition that relaxes or disables an otherwise applicable rule. |

A **SHOULD** or **SHOULD NOT** MUST NOT be treated as an unconditional requirement. Deviation MAY occur only when supported by repository evidence, an explicit requirement, a verified technical constraint, or a clearly identified tradeoff. Convenience, speculation, preference, and hypothetical future requirements are not sufficient justification.

For a conditional rule:

1. Establish whether its **TRIGGER** is satisfied by evidence.
2. If the trigger is not established, do not activate the rule automatically.
3. If the trigger is established, enforce its normative requirements.
4. Apply an **EXCEPTION** only when every condition stated by that exception is satisfied.

Missing information establishes neither a trigger nor an exception.

An **EXCEPTION** MUST be explicit, address a concrete requirement or constraint, be scoped to the smallest affected behavior, preserve unrelated requirements, and remain distinguishable from normal behavior. A generic environment or mode MUST NOT automatically disable unrelated engineering or security requirements.

## 2. Rule Application

Apply `references/engineering-guidelines.md` to every coding task.

Apply `references/security-guidelines.md` when a task changes or introduces a trust boundary or security-sensitive behavior, including authentication, authorization, credentials, externally controlled input, externally reachable APIs, server-side network access, files, serialization, dynamic execution, cryptography, sensitive data, privileged operations, dependencies, build security, artifact integrity, or deployment security.

Apply `references/java-guidelines.md` when Java, Kotlin, or related JVM technologies are involved.

Apply `references/project-guidelines.md` when discovering, establishing, evaluating, or modifying repository-specific engineering rules.

A more specific applicable rule refines a more general rule. Repository-specific rules MUST be established from project evidence rather than inferred from a single implementation. Existing insecure or incorrect behavior MUST NOT be treated as sufficient justification to weaken a general correctness or security requirement.

### Evidence

Engineering conclusions MUST be supported by available evidence. Relevant evidence includes source code and actual definitions, callers and callees, tests and runtime behavior, build and analysis configuration, framework configuration, schemas and public contracts, logs and reproducible failures, repository documentation, and multiple consistent current implementations.

Verified facts MUST be distinguished from assumptions. An unverified assumption MUST NOT be presented as a confirmed root cause, contract, security property, performance characteristic, or project convention.

When a decision depends on missing information, inspect available code, configuration, tests, definitions, documentation, and runtime evidence before deciding. If the missing fact can be established from available evidence, establish it before making the material engineering decision. If it cannot be established, do not invent it.

### Change Decision Gate

Before introducing any of the following, establish its concrete present-day justification:

| Change | Required justification |
|---|---|
| New abstraction or interface | Real variation point, architectural boundary, meaningful decoupling, or duplicated knowledge |
| New dependency | Existing stack cannot adequately satisfy the requirement |
| Compatibility behavior | Verified consumer, persisted data, protocol, or explicit compatibility requirement |
| Fallback or degradation | Verified failure mode and defined fallback semantics |
| Retry | Verified retryable failure and safe or idempotent execution |
| Public API | Actual consumer or explicit contract |
| Security exception | Applicable explicit **EXCEPTION** with all conditions satisfied |
| Performance optimization | Requirement, expected scale, profiling, benchmark, or runtime evidence |

Hypothetical future requirements MUST NOT justify additional architecture.

## 3. Engineering Workflow

Before making a material behavior-changing decision:

1. Establish the requested behavior and affected scope.
2. Identify the affected language, framework, module, boundary, and contract when relevant.
3. Load the applicable reference guidelines.
4. Inspect relevant repository-specific engineering rules and configuration.
5. Inspect the affected implementation and actual API or type definitions.
6. Inspect relevant callers, callees, tests, and analogous implementations.
7. Separate verified facts from assumptions.
8. Determine the smallest change that satisfies the verified requirement.

Unfamiliar external APIs MUST be verified from authoritative documentation, dependency source, or established project usage before introduction. Internal methods MUST be inspected at their actual definition before their behavior or signature is assumed.

Changes MUST remain focused on the requested engineering scope. Unrelated cleanup, formatting, modernization, or refactoring MUST NOT be included merely because an issue was discovered nearby.

## 4. Task Workflows

### Bug Investigation

The investigation MUST:

1. Establish the observed incorrect behavior.
2. Reproduce it when a practical reproduction path exists.
3. Trace the actual execution path.
4. Gather evidence.
5. Form a falsifiable root-cause hypothesis.
6. Verify the hypothesis before declaring the root cause.
7. Establish the violated contract.
8. Fix the verified cause with the smallest scoped change.
9. Add regression coverage when practical.
10. Re-run the reported scenario and relevant tests.

Suspicious-looking code, correlation, or an untested hypothesis MUST NOT be reported as a confirmed root cause. A callee contract MUST NOT be broadened, validation weakened, or fallback behavior introduced merely because one caller violates an established contract.

### Feature Implementation

Feature design MUST establish the owning domain concept, responsible component, affected contract, closest existing implementation, and existing extension point when one exists.

Existing abstractions SHOULD be reused when they correctly represent the required behavior. A new abstraction MUST have a concrete present-day responsibility, boundary, variation point, decoupling benefit, or duplicated knowledge to justify it. Public API surface MUST be limited to actual consumers and verified contracts.

New business behavior MUST have corresponding verification.

### Refactoring

Refactoring MUST have a concrete motivation such as duplicated business knowledge, misplaced responsibility, excessive coupling, a violated dependency boundary, or demonstrated cognitive complexity.

Externally observable behavior MUST remain unchanged unless behavior change is part of the verified requirement. Refactoring MUST NOT introduce parallel replacement variants such as `XxxV2`, `newXxx()`, or `XxxImpl2` instead of replacing obsolete behavior.

Patterns, stylistic preference, arbitrary metrics, or hypothetical extensibility MUST NOT be sufficient reasons for refactoring.

### Code Review

Review findings MUST be evidence based and prioritized by impact:

1. Correctness and data integrity.
2. Security, authentication, authorization, and trust-boundary violations.
3. Concurrency and resource safety.
4. Contract and compatibility correctness.
5. Missing or insufficient tests and verification.
6. Responsibility and dependency boundaries.
7. Maintainability.
8. Performance problems supported by evidence.

Code smells are investigation signals, not automatic defects. A substantive finding MUST identify concrete evidence and explain the actual or likely impact.

### Security-Sensitive Changes

**TRIGGER:** `references/security-guidelines.md` applies to the affected behavior.

The change MUST identify the relevant trust boundary, protected asset or operation, untrusted actor or input, and security property being enforced. Existing framework or infrastructure controls MUST be verified before they are relied upon or duplicated.

Applicable security **TRIGGER**, requirement, and **EXCEPTION** rules MUST be evaluated from evidence. Relevant allowed behavior and denied or failure behavior MUST be verified.

Following a guideline or passing tests MUST NOT be presented as proof that an implementation is comprehensively secure.

## 5. Verification

Code changes MUST be verified before they are considered complete.

Verification MUST include the most focused relevant tests and SHOULD include broader regression, build, compiler, static-analysis, or runtime checks when required by the affected boundary. Failures MUST be investigated before being classified as unrelated.

A failing test MUST NOT be changed merely to make an implementation pass. Before changing an existing failing test, establish whether production behavior is wrong, the test is wrong, or the contract intentionally changed.

The final change set MUST be inspected for accidental or unrelated modifications. Temporary debugging output, placeholders, incomplete TODOs, and other intermediate implementation artifacts MUST NOT remain in completed code.

Security-sensitive changes MUST verify relevant failure paths in addition to successful behavior.

Verification that cannot be performed MUST remain explicitly unverified rather than being inferred from inspection alone.
