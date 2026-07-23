---
name: security-and-hardening
description: Hardens backend code against vulnerabilities. Use when handling user input, authentication, data storage, or external integrations. Use when building any feature that accepts untrusted data, manages user sessions, or interacts with third-party services.
---

# Security and Hardening

## Overview

Treat every external input as hostile, every secret as sacred, and every authorization check as mandatory. Security is a constraint on every line of code that touches user data, authentication, or external systems — not a phase you get to later.

## When to Use

- Building anything that accepts user input
- Implementing authentication or authorization
- Storing or transmitting sensitive data
- Integrating with external APIs or services
- Adding file uploads, webhooks, or callbacks
- Handling payment or PII data

## Process: Threat Model First

Before hardening, spend five minutes thinking like an attacker:

1. **Map the trust boundaries** — where does untrusted data cross into your system? HTTP requests, form fields, file uploads, webhooks, third-party APIs, message queues, LLM output.
2. **Name the assets** — credentials, PII, payment data, admin actions, money movement.
3. **Run STRIDE over each boundary:**

| Threat | Ask | Typical mitigation |
|---|---|---|
| **S**poofing | Can someone impersonate a user/service? | Authentication, signature verification |
| **T**ampering | Can data be altered in transit or at rest? | Integrity checks, parameterized queries, TLS |
| **R**epudiation | Can an action be denied later? | Audit logging of security events |
| **I**nformation disclosure | Can data leak? | Encryption, field allowlists, generic errors |
| **D**enial of service | Can it be overwhelmed? | Rate limiting, input size caps, timeouts |
| **E**levation of privilege | Can a user gain rights they shouldn't? | Authorization checks, least privilege |

If you can't name the trust boundaries for a feature, you're not ready to secure it.

## The Three-Tier Boundary System

**Always do:** validate all external input at the boundary (controller/handler); parameterize all queries — never concatenate user input into SQL/JPQL; hash passwords with BCrypt/Argon2; use TLS everywhere; set security headers (CSP, HSTS, X-Frame-Options); use httpOnly/secure/sameSite session cookies; run the dependency audit before every release.

**Ask first:** new auth flows, new categories of sensitive data, new external integrations, CORS changes, file upload handlers, rate-limit changes, elevated roles.

**Never do:** commit secrets to version control; log sensitive data (passwords, tokens, full card numbers); trust client-side validation as a security boundary; disable security headers for convenience; expose stack traces to users.

## Prevention Patterns

### Injection (SQL)

```java
// BAD: string concatenation
String sql = "SELECT * FROM users WHERE id = '" + userId + "'";

// GOOD: parameterized query
PreparedStatement stmt = connection.prepareStatement("SELECT * FROM users WHERE id = ?");
stmt.setString(1, userId);
```

If you're using an ORM (JPA/Hibernate, jOOQ, MyBatis...), the same rule applies: build queries through the library's parameter binding, never by concatenating a raw string.

### Authentication

```java
String hash = BCrypt.hashpw(plaintext, BCrypt.gensalt(12));
boolean valid = BCrypt.checkpw(plaintext, hash);
```

Session cookies: `httpOnly`, `secure`, `sameSite=Lax`, bounded expiry, secret from environment/vault — never in code.

### Broken Access Control

```java
// Handler for PATCH /api/tasks/{id}
public Response updateTask(String taskId, User authenticatedUser, TaskUpdate body) {
    Task task = taskService.findById(taskId);
    if (!task.getOwnerId().equals(authenticatedUser.getId())) {
        throw new ForbiddenException("Not authorized to modify this task");
    }
    return Response.ok(taskService.update(taskId, body));
}
```

Check authorization, not just authentication, on every protected endpoint — regardless of which web framework wires the route to this handler.

### Security Misconfiguration (Headers & CORS)

Set these on every response, at the framework's filter/middleware layer rather than per-handler: `Content-Security-Policy` (start from `default-src 'self'`), `Strict-Transport-Security`, `X-Frame-Options`, `X-Content-Type-Options`. For CORS, allow only known origins read from configuration (`System.getenv("ALLOWED_ORIGINS")`, split on comma) — never use a wildcard (`*`) origin alongside credentials.

### Sensitive Data Exposure

```java
// Never return the entity directly — project to a DTO that excludes secrets
public record PublicUser(String id, String email, String displayName) {}
```

Read secrets (API keys, DB credentials) from environment variables or a secrets manager — never hardcode them.

### Server-Side Request Forgery (SSRF)

Any server-side fetch of a user-influenced URL (webhooks, "import from URL", link previews) can be aimed at internal services (cloud metadata, `localhost`, private IPs).

```java
URL url = new URL(rawUrl);
if (!url.getProtocol().equals("https")) throw new IllegalArgumentException("https only");
if (!ALLOWED_HOSTS.contains(url.getHost())) throw new IllegalArgumentException("host not allowed");
for (InetAddress addr : InetAddress.getAllByName(url.getHost())) {
    if (addr.isLoopbackAddress() || addr.isLinkLocalAddress() || addr.isSiteLocalAddress()) {
        throw new IllegalArgumentException("private/reserved IP");
    }
}
```

Note the TOCTOU gap: a short-TTL DNS record can rebind between validation and connection. For high-risk surfaces, resolve once and connect to the pinned IP.

## Input Validation

Validate at the boundary with a declarative schema (Jakarta Bean Validation annotations, or your framework's equivalent), not ad hoc checks scattered through the service layer:

```java
public record CreateTaskRequest(
    @NotBlank @Size(max = 200) String title,
    @Size(max = 2000) String description,
    @NotNull TaskPriority priority
) {}

// Handler validates the incoming request against the schema before it reaches the service layer
public Response createTask(CreateTaskRequest req) {
    validator.validate(req); // throws on constraint violation
    return Response.created(taskService.create(req));
}
```

**File uploads:** restrict allowed content types and max size; don't trust the file extension — check magic bytes for anything security-sensitive.

## Dependency & Supply-Chain Hygiene

- Run the build tool's dependency audit before release (`mvn org.owasp:dependency-check:check`, or Gradle's `dependencyCheckAnalyze`). A critical/high finding reachable in your runtime path blocks the release; unreachable or dev-only findings can be tracked and fixed on the next cycle.
- Pin dependency versions; review the lockfile/BOM diff on every bump, not just the version number in `pom.xml`/`build.gradle`.
- Upgrade one dependency per change where practical — a bulk "bump deps" that breaks the build hides which package did it.
- Audits only catch known advisories, not a newly malicious or typosquatted package (e.g. a plausible-looking group ID). Review new dependencies for ownership, maintenance, and release age before adding them.

## Rate Limiting

Use a token-bucket limiter (Bucket4j, Resilience4j, or your gateway) with a stricter limit on auth endpoints than on general API traffic.

## Secrets Management

Keep secrets in environment variables, a vault, or your platform's secret store — never in `application.properties`/`application.yml` checked into git. Add `.env`, `*.pem`, `*.key`, and any local override files to `.gitignore`. If a secret is ever committed, rotate it immediately — deleting the line or rewriting history is not enough, assume it's compromised the moment it reaches a remote.

## Securing AI / LLM Features

If your service calls an LLM (chatbot, summarizer, agent, RAG), it inherits a new attack surface — map it to the [OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/):

- **Treat model output as untrusted input.** Never pass it straight into a query, a shell command, or a file path.
- **Assume prompts can be hijacked.** Untrusted text in the context (a user message, a fetched document) can carry instructions; enforce permissions in code, not in the system prompt.
- **Keep secrets and other tenants' data out of prompts** — anything in context can be echoed back.
- **Constrain tool/agent permissions** to the minimum, and require confirmation for destructive actions.
- **Bound consumption** — cap tokens, request rate, and recursion depth.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "This is an internal tool, security doesn't matter" | Internal tools get compromised. Attackers target the weakest link. |
| "We'll add security later" | Retrofitting is far harder than building it in. Add it now. |
| "The framework handles security" | Frameworks provide tools, not guarantees. You still have to use them correctly. |
| "Threat modeling is overkill here" | Five minutes of "how would I attack this?" prevents design flaws no control can patch later. |
| "The audit passed, so the dependency is safe" | Audits match known advisories; they don't catch a newly malicious package. |

## Red Flags

- User input passed directly into a query, a shell command, or a file path
- Secrets in source code or commit history
- Endpoints without authentication or authorization checks
- Wildcard CORS origins
- No rate limiting on authentication endpoints
- Stack traces exposed to callers
- Dependencies with known critical/high vulnerabilities, unreviewed
- Server fetches a user-supplied URL without an allowlist (SSRF)
- LLM output passed into a query, shell, or file path without validation

## Verification

After implementing security-relevant code:

- [ ] No secrets in source code or git history
- [ ] All external input validated at the boundary
- [ ] Authentication and authorization checked on every protected endpoint
- [ ] Dependency audit has no unmitigated reachable critical/high findings
- [ ] Server-side URL fetches validated against an allowlist (no SSRF)
- [ ] Error responses don't expose internal details
- [ ] LLM output validated before use (if AI features present)
