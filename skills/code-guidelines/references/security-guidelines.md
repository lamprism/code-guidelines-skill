# Security Guidelines

These requirements apply when the security reference is triggered by the skill. They use the normative model defined by `../SKILL.md` and supplement `engineering-guidelines.md`.

Applicable OWASP guidance, especially the OWASP Top 10 and OWASP Cheat Sheet Series, SHOULD be used as a security baseline. Security controls MUST still be derived from the actual trust boundary, asset, attacker capability, and verified system contract rather than copied mechanically from a checklist.

## 1. Security Boundaries

- Security MUST be treated as part of system design rather than post-implementation cleanup.
- Data crossing an untrusted boundary MUST be treated as untrusted until validated according to the receiving contract.
- Client-side validation, UI restrictions, network location, obscurity, or validation performed only by an untrusted client MUST NOT be treated as server-side security controls.
- Security-sensitive failures MUST fail closed unless an explicit applicable **EXCEPTION** defines different behavior.
- Least privilege MUST be applied to users, services, credentials, filesystem access, network access, database permissions, and infrastructure identities.
- Established platform, framework, and mature security-library controls SHOULD be used instead of custom authentication protocols, password hashing, cryptographic primitives, session mechanisms, token signing, sanitizers, or equivalent security mechanisms.
- Security-sensitive APIs and defaults MUST be verified from authoritative documentation or implementation evidence before they are changed or relied upon.

## 2. Identity And Access

### Authentication

**TRIGGER:** A protected operation requires caller identity.

- Authentication MUST be enforced at a trusted server or service boundary.
- Authoritative identity MUST come from the established authentication context rather than caller-provided user, role, account, or tenant fields.
- Plaintext or reversibly encoded passwords MUST NOT be stored.
- Passwords MUST use the project's established password-hashing mechanism and an algorithm designed for password hashing.
- General-purpose fast hashes MUST NOT be used directly for password storage.
- Account recovery, verification, and similar security tokens MUST be unpredictable, scoped to their intended operation, time bounded when expiry is part of the protocol, and invalidated according to the verified lifecycle.

### Authorization

**TRIGGER:** An operation accesses or changes a protected resource or privileged capability.

- Authorization MUST deny by default.
- Authentication alone MUST NOT imply authorization.
- Authorization MUST be checked for the requested action and protected resource at a trusted boundary.
- UI visibility, route hiding, client-side checks, or possession of a resource identifier MUST NOT be treated as authorization.
- Tenant, organization, project, account, ownership, and role boundaries MUST use authoritative server-side context.

### Object-Level Authorization

**TRIGGER:** A protected operation selects a resource using an identifier directly or indirectly controlled by an untrusted caller.

- The authenticated principal MUST be authorized for the selected resource and requested action.
- Caller-provided owner, tenant, account, or role fields MUST NOT be treated as proof of authorization.

**EXCEPTION:** A separate per-object check MAY be omitted when the resource is explicitly public by contract or when a verified upstream capability has already constrained access to the authorized resource and operation.

### Sessions, Cookies, And Tokens

**TRIGGER:** The application uses sessions, cookies, bearer tokens, or signed identity tokens.

- Framework-managed session and cookie mechanisms SHOULD be used when available.
- Session identifiers and secret bearer credentials MUST use cryptographically secure generation or a trusted identity system.
- Authentication cookies MUST use security attributes appropriate to the verified browser architecture, including `Secure`, `HttpOnly`, and `SameSite` when applicable.
- Sensitive credentials and long-lived secrets MUST NOT be placed in URLs or query parameters.
- Signed tokens MUST be verified, not merely decoded.
- Applicable signature, expiration, issuer, audience, and other security-relevant claims MUST be validated according to the token contract.
- Signature verification, certificate verification, or hostname verification MUST NOT be disabled in production behavior.

## 3. Input, Injection, And Output

### External Input Validation

**TRIGGER:** Data controlled by an external caller, external system, uploaded content, message, webhook, imported file, or other untrusted source crosses into trusted behavior.

- Input MUST be validated against the receiving contract before security-sensitive use.
- Validation MUST cover applicable type, structure, length, range, allowed values, relationships, and business invariants.
- Allowlist validation SHOULD be used when the valid input space is bounded.
- Denylist filtering MUST NOT be the primary security control when valid input can be defined.
- Security-sensitive checks MUST operate on a canonical representation when equivalent encodings or representations can alter the decision.
- Externally controlled payload size and complexity MUST be bounded when unbounded input can exhaust CPU, memory, storage, parser resources, or network capacity.

### Injection

**TRIGGER:** Untrusted data contributes to a query, command, expression, template, interpreter input, or other executable syntax.

- A structured or parameterized API MUST be used when one exists.
- Untrusted values MUST NOT be concatenated into executable SQL, shell commands, LDAP filters, XPath expressions, template expressions, or equivalent executable syntax.
- Structural elements that cannot be parameterized MUST be selected from a trusted allowlist.
- Escaping MUST NOT replace parameterization when a structured parameter API exists.
- Direct process APIs with structured argument lists SHOULD be used instead of invoking a command shell when a shell is unnecessary.

### Browser Output And XSS

**TRIGGER:** Untrusted content is rendered into a browser-controlled execution context.

- Output MUST be encoded for its actual destination context.
- Framework contextual escaping SHOULD remain enabled.
- One generic escaping function MUST NOT be assumed safe for HTML text, attributes, URLs, JavaScript, and CSS simultaneously.
- Rich or active content that must be accepted MUST use a mature sanitizer configured for the required content model.
- Regular expressions MUST NOT be used as a general HTML sanitizer.
- Content Security Policy MAY provide defense in depth but MUST NOT replace correct output encoding and safe DOM usage.

### Mass Assignment

**TRIGGER:** An external request can bind fields to an internal model that contains fields not intentionally writable by that operation.

- Explicit request DTOs, commands, or allowed-field mappings MUST define writable fields.
- Server-owned identity, ownership, role, tenant, price, approval, audit, and security state MUST NOT become writable merely because matching fields appear in a client payload.

## 4. Web And Network Security

### CSRF

**TRIGGER:** A browser state-changing request relies on credentials that the browser attaches automatically.

- CSRF protection MUST be provided by the framework or an equivalent verified mechanism.
- CSRF protection MUST NOT be disabled merely to make a request succeed.

**EXCEPTION:** Additional CSRF protection MAY be omitted when the verified authentication and request model does not rely on automatically attached browser credentials and therefore does not expose the relevant CSRF attack path.

### CORS

**TRIGGER:** Browser clients from another origin require access to an HTTP endpoint.

- Allowed origins MUST correspond to verified consumers.
- Arbitrary request origins MUST NOT be reflected as trusted origins.
- Wildcard origins MUST NOT be combined with credentialed browser access.
- Only required methods and headers SHOULD be allowed.
- CORS MUST NOT be treated as authentication or authorization.

**EXCEPTION:** An explicit local development origin MAY use plain HTTP when it is narrowly scoped to local development and cannot become active in production unintentionally. A generic development flag MUST NOT automatically enable unrestricted CORS.

### HTTP Security

**TRIGGER:** The application exposes HTTP behavior.

- Sensitive or authenticated traffic MUST use HTTPS outside an explicit local-development **EXCEPTION**.
- TLS certificate and hostname verification MUST NOT be disabled in production behavior.
- State-changing operations MUST NOT rely on `GET`.
- Detailed stack traces, SQL statements, internal paths, credentials, secrets, and unnecessary framework diagnostics MUST NOT be exposed in HTTP error responses.
- Security headers SHOULD be configured when relevant to the browser architecture.
- Request bodies, headers, uploads, and other externally controlled protocol resources MUST have limits when unbounded consumption is possible.

### Redirects

**TRIGGER:** An untrusted caller can influence a redirect or forward destination.

- Arbitrary untrusted destinations MUST NOT be used directly.
- Opaque destination identifiers mapped to trusted destinations SHOULD be preferred.
- When external redirects are required, scheme, host, port, and other security-relevant properties MUST be constrained to the verified destination policy.
- Prefix, suffix, or substring checks MUST NOT be the sole security-sensitive URL validation mechanism.

### Server-Side Requests

**TRIGGER:** Untrusted input directly or indirectly influences the effective destination of a network request made by the application.

- The effective destination MUST be constrained according to the verified outbound-network policy before the request is made.
- Trusted destination allowlisting SHOULD be used when the valid destination set is bounded.
- Allowed schemes MUST be constrained.
- Internal, loopback, link-local, metadata, administrative, and other prohibited destinations MUST be rejected unless explicitly required by the verified contract.
- Validation MUST account for parsing, resolution, and the effective destination.
- DNS behavior and rebinding MUST be evaluated when hostname resolution can change whether a destination is permitted.
- Redirects MUST NOT allow a permitted destination to escape into a prohibited destination.
- Network timeouts, redirect limits, and response-size limits MUST be bounded when controlled by an untrusted destination.
- String prefix, suffix, substring, or blacklist checks MUST NOT be the sole SSRF control.

**EXCEPTION:** Caller-input destination validation is not required when the destination is selected entirely from trusted server-side configuration and untrusted input cannot influence its effective network destination.

## 5. Files, Serialization, And Dynamic Execution

### File Uploads

**TRIGGER:** The application accepts files from an untrusted source.

- File size MUST be bounded.
- Accepted file types MUST be limited to the business requirement when the set is bounded.
- Client-provided filename, extension, `Content-Type`, and MIME metadata MUST NOT be trusted as proof of file type.
- Server-controlled storage names SHOULD be used when the original filename is not required as storage identity.
- Uploaded content MUST NOT become executable server-side code.
- Upload, replace, read, and delete operations MUST enforce their own authorization requirements when protected.
- Storage and serving locations MUST reflect the verified execution and exposure model.

### Paths And Archives

**TRIGGER:** Untrusted input influences a filesystem path or archive extraction.

- Untrusted input MUST NOT become an unrestricted filesystem path.
- Resolved paths MUST remain within the intended root after applicable normalization and resolution.
- Removing traversal substrings such as `../` MUST NOT be the sole path-security control.
- Archive entries MUST be validated before extraction and MUST NOT escape the extraction directory.
- Untrusted archives MUST have bounded entry count, expanded size, nesting, processing time, or equivalent resource controls when those resources can be exhausted.

### Deserialization

**TRIGGER:** Untrusted data is deserialized into runtime objects.

- Arbitrary runtime object graphs MUST NOT be constructed from untrusted data.
- Explicit DTOs or schemas SHOULD be used for cross-boundary data.
- Unrestricted polymorphic deserialization MUST NOT be enabled for untrusted input.
- Externally supplied class or type names MUST NOT control arbitrary class instantiation.
- Deserialized data MUST still be validated before security-sensitive use.

### XML

**TRIGGER:** Untrusted XML is parsed.

- External entity resolution and external resource retrieval MUST be disabled unless explicitly required by the verified contract.
- DTD processing SHOULD be disabled when not required.
- Parser resource limits MUST be configured when untrusted input can cause excessive expansion or processing.
- Parser security defaults MUST be verified rather than assumed.

### Dynamic Execution

**TRIGGER:** Untrusted input can influence source code, expressions, templates, scripts, dynamic class loading, or equivalent executable behavior.

- Untrusted input MUST NOT be evaluated directly as executable code or unrestricted expressions.
- Untrusted selections SHOULD be mapped to trusted predefined operations.
- A claimed sandbox MUST NOT be treated as a security boundary without verification of its actual guarantees.

## 6. Cryptography, Secrets, And Sensitive Data

### Cryptography

- Custom cryptographic algorithms or protocols MUST NOT be introduced.
- Established cryptographic libraries and algorithms MUST be used for security-sensitive cryptography.
- Obsolete or broken algorithms MUST NOT be used for security-sensitive purposes.
- ECB mode MUST NOT be used for confidential data.
- Authenticated encryption SHOULD be used when both confidentiality and integrity are required.
- Keys, nonces, IVs, and salts MUST satisfy the selected algorithm's requirements.
- Cryptographic keys MUST NOT be hard-coded in production source code.

### Secure Randomness

**TRIGGER:** A value acts as a secret credential, unpredictable token, nonce, session identifier, reset token, verification token, or equivalent security value.

- A cryptographically secure random generator or trusted identity system MUST be used.
- Ordinary pseudo-random generators, timestamps, counters, or other predictable values MUST NOT be used as secret credentials.

### Secrets

- Production secrets, passwords, API keys, private keys, access tokens, and equivalent credentials MUST NOT be hard-coded in source code.
- Secrets SHOULD be obtained from the project's approved secret-management or protected configuration mechanism.
- Secrets MUST NOT be placed in URLs, source-control history, example configuration, logs, metrics, analytics events, snapshots, or exception messages.
- Reusable credentials MUST NOT be logged even under a development or diagnostic **EXCEPTION**.
- Test credentials MUST NOT provide access to production systems or production data.

### Sensitive Data

**TRIGGER:** The system handles data classified as sensitive by the verified product, security, privacy, or regulatory contract.

- Collection, transfer, storage, logging, and retention MUST be limited to the verified requirement.
- Sensitive data MUST use secure transport when crossing an untrusted network.
- At-rest protection MUST follow the verified sensitivity, threat model, regulatory requirement, or project security contract.
- Internal models containing non-public fields MUST NOT be exposed automatically across public boundaries.
- Production secrets or unnecessary personal data MUST NOT be copied into tests, fixtures, examples, or development logs.

## 7. Availability, Concurrency, And Abuse

### Abuse Protection

**TRIGGER:** An externally reachable operation can be repeatedly invoked to guess credentials, enumerate protected resources, send repeated user-visible actions, create significant cost, consume scarce resources, or otherwise produce a verified abuse path.

- The operation MUST have a bounded abuse-control strategy.
- The strategy MAY use rate limiting, quotas, throttling, lockouts, account-level controls, or an existing platform mechanism.
- The control MUST be enforced at a boundary capable of constraining the relevant actor or resource.

**EXCEPTION:** Additional application-level control MAY be omitted when a verified infrastructure control already enforces the required bound for the exact operation and relevant identity or resource scope.

### Resource Exhaustion

**TRIGGER:** An untrusted actor can control input size, collection size, recursion, parser depth, query complexity, execution duration, memory growth, decompression, or another resource whose cost can become unbounded.

- The controllable resource MUST be bounded.
- Streaming or bounded processing SHOULD be used instead of loading arbitrarily large content into memory when applicable.
- External and potentially blocking operations MUST have timeouts.
- Regular expressions used on untrusted input MUST NOT have uncontrolled catastrophic backtracking behavior.

### Concurrency And Integrity

**TRIGGER:** Correctness or security depends on mutable state remaining consistent across concurrent operations.

- Security and integrity decisions MUST remain correct under the actual concurrency model.
- Application-level pre-checks MUST NOT be the sole protection for invariants that require atomic enforcement.
- Transactions, constraints, atomic operations, locks, compare-and-set, or equivalent mechanisms MUST be used when the invariant requires atomicity.
- Single-use security tokens and idempotency or replay controls MUST enforce uniqueness atomically when concurrent use is possible.

## 8. Supply Chain And Deployment

### Dependencies

**TRIGGER:** A change introduces or upgrades a third-party dependency, plugin, action, build extension, executable artifact, or remote installation mechanism.

- The existing standard library and dependency set MUST be checked before adding a new dependency.
- Dependencies SHOULD come from authoritative, actively maintained, and trusted sources.
- Known vulnerabilities relevant to the introduced version MUST be evaluated when dependency security is part of the change.
- Existing lockfile, checksum, signature, artifact-repository, or dependency-verification mechanisms MUST NOT be bypassed merely to make resolution succeed.
- Remote installation scripts MUST NOT be executed without understanding and trusting their source and behavior.
- Unrelated dependency upgrades MUST NOT be included in a focused change unless required for correctness or security.

### Build And CI/CD

**TRIGGER:** A change affects build scripts, CI workflows, package-manager hooks, generators, release automation, or deployment behavior.

- CI credentials and workflow permissions MUST use least privilege.
- Secrets MUST NOT be exposed to untrusted build or pull-request code without an explicit secure design.
- Third-party actions, plugins, and build extensions MUST be treated as executable dependencies.
- Branch, artifact, signature, deployment, or integrity protections MUST NOT be weakened merely to make a pipeline pass.

### Configuration And Deployment

**TRIGGER:** A change affects production security configuration or externally reachable deployment behavior.

- Default passwords, sample credentials, development secrets, and test keys MUST NOT ship as production credentials.
- Debug endpoints, development consoles, authentication bypasses, verbose exception pages, or insecure administrative interfaces MUST NOT be enabled in production behavior.
- Missing mandatory production security configuration MUST fail clearly rather than silently selecting an insecure default.
- Development-only relaxed behavior MUST be explicitly scoped and MUST NOT become an implicit production fallback.

## 9. Security Verification

Security-sensitive behavior MUST be verified at the boundary where the security property is enforced.

- Authorization tests MUST include relevant allowed and denied cases.
- Object-level authorization SHOULD verify cross-identity denial when resources are identity scoped.
- Authentication tests SHOULD cover relevant invalid, expired, revoked, malformed, or otherwise unacceptable credentials.
- Validation tests MUST cover meaningful invalid boundaries when the change introduces validation.
- Security-token tests MUST cover applicable expiration, scope, replay, single-use, signature, or claim behavior.
- Failure-path tests MUST verify that security-control failures do not silently permit the protected operation.
- Security tests MUST NOT be weakened merely to accommodate insecure production behavior.
- A verified vulnerability fix SHOULD include regression coverage for the vulnerable behavior when a practical test path exists.
- Static analysis, dependency scanning, secret scanning, and similar tools SHOULD be used when the repository already provides them or the task requires them.

Successful tests or tool output MUST NOT be treated as proof that the application is comprehensively secure.

## 10. Security Review Signals

The following patterns are investigation signals, not automatic proof of a vulnerability.

| Signal | Concern |
|---|---|
| Caller-provided user, tenant, owner, or role identity | Authorization may rely on untrusted identity data |
| Protected resource lookup directly from request ID | Object-level authorization may be missing |
| String-built SQL or executable query | Injection may be possible |
| Shell command construction | Command injection may be possible |
| Raw HTML insertion or disabled escaping | Cross-site scripting may be possible |
| Disabled CSRF protection | Browser-authenticated state changes may be forgeable |
| Wildcard or reflected CORS | Untrusted origins may receive unintended access |
| Caller-controlled server-side URL | SSRF may expose internal services or metadata |
| Caller-controlled redirect | Open redirect or boundary bypass may be possible |
| Caller-controlled filesystem path | Path traversal or arbitrary file access may be possible |
| Archive extraction | Path escape or decompression exhaustion may be possible |
| Native or unrestricted polymorphic deserialization | Unsafe object construction may be possible |
| XML parser with unverified defaults | External entity or resource processing may be enabled |
| `eval` or equivalent dynamic execution | Untrusted input may become executable |
| Custom cryptography | Cryptographic design flaws may exist |
| Fast hash used for passwords | Password storage may be weak |
| Ordinary randomness used for security tokens | Credentials may be predictable |
| Hard-coded secret or secret in logs | Reusable credentials may leak |
| Disabled TLS verification | Network authentication may be bypassed |
| Token decoded without verification | Untrusted claims may be accepted |
| Broad fallback around authorization | Security failure may become implicit access |
| Default-allow security branch | Unknown states may gain privileges |
| Unbounded externally controlled resource | Resource exhaustion may be possible |
| Check followed by separate mutable-state update | Race or TOCTOU failure may be possible |
| External DTO bound directly to privileged model | Mass assignment may expose protected fields |
| Public debug or management endpoint | Sensitive operations or data may be exposed |
| Unfamiliar executable dependency or remote script | Supply-chain risk may be introduced |
| Security test weakened with production change | Verification may have been adapted to insecure behavior |

**TRIGGER:** A security review signal is found in relevant code.

- Reachability by an untrusted actor MUST be established or explicitly left unverified.
- The affected asset or security property MUST be identified.
- Actual framework, protocol, and deployment behavior SHOULD be inspected.
- Existing mitigations MUST be verified before they are relied upon.
- Exploitability or failure behavior SHOULD be verified when practical and safe.
- A potential risk MUST NOT be reported as a confirmed vulnerability without supporting evidence.

## 11. Exception Boundaries

Security controls MUST be proportional to the verified threat model, but convenience MUST NOT create implicit exceptions.

Authentication MUST NOT be added to intentionally public resources solely because the security reference is loaded. Encryption MUST NOT be added to data with no verified confidentiality requirement. Application-level rate limiting SHOULD NOT duplicate an exact verified infrastructure control. Sanitization SHOULD NOT replace contextual output encoding. Security wrappers SHOULD NOT duplicate already verified safe framework behavior without concrete benefit.

A development, test, debug, or local environment MAY relax a rule only through an explicit applicable **EXCEPTION**. The relaxation MUST be narrowly scoped, MUST NOT expose production secrets or sensitive production data, and MUST NOT become active in production unintentionally.

Compliance with this document MUST NOT be used as the sole basis for claiming that code is secure.
