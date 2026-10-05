---
name: security-and-hardening
description: Hardens backend code against vulnerabilities. Use when building anything that accepts untrusted input, handles authentication or sensitive data, adds or upgrades dependencies, or calls external services or LLMs.
---

# Security and Hardening

## Overview

Treat every external input as hostile, every secret as sacred, and every authorization check as mandatory. Security is a constraint on every line that touches user data, authentication, or external systems — not a phase you get to later.

## When to Use

- Accepting user input, file uploads, webhooks, or callbacks
- Implementing authentication or authorization
- Storing or transmitting sensitive data (credentials, PII, payment data)
- Integrating with external APIs, message queues, or LLMs
- Adding or upgrading a dependency

## Process: Threat Model First

Before hardening, spend five minutes thinking like an attacker:

1. **Map the trust boundaries** — where does untrusted data cross into the system? HTTP requests, uploads, webhooks, third-party APIs, message queues, LLM output.
2. **Name the assets** — credentials, PII, payment data, admin actions, money movement.
3. **Run STRIDE over each boundary:** **S**poofing (authn, signatures) · **T**ampering (integrity, parameterized queries, TLS) · **R**epudiation (audit log security events) · **I**nformation disclosure (DTO allowlists, generic errors) · **D**enial of service (rate limits, size caps, timeouts) · **E**levation of privilege (authz checks, least privilege).

If you can't name the trust boundaries for a feature, you're not ready to secure it.

## The Three-Tier Boundary System

**Always:** validate external input at the boundary with a declarative schema (Bean Validation or equivalent); parameterize every query, including ORM/JPQL — never concatenate input; check *authorization* (ownership/role), not just authentication, on every protected endpoint; return DTOs, never entities; hash passwords with BCrypt/Argon2; set security headers and CORS at the filter layer; use `httpOnly`/`secure`/`sameSite` session cookies; rate-limit auth endpoints more strictly than general traffic.

**Ask first:** new auth flows, new categories of sensitive data, new external integrations, CORS changes, file upload handlers, rate-limit changes, elevated roles.

**Never:** commit secrets (including in `application.yml`); log passwords, tokens, or full card numbers; trust client-side validation; use a wildcard CORS origin with credentials; expose stack traces to callers; trust a file's extension (check type, size, and magic bytes).

## Easy-to-Miss Risks

### Server-Side Request Forgery (SSRF)

Any server-side fetch of a user-influenced URL (webhooks, "import from URL", link previews) can be aimed at internal services — cloud metadata, `localhost`, private IPs.

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

Mind the TOCTOU gap: a short-TTL DNS record can rebind between validation and connection. For high-risk surfaces, resolve once and connect to the pinned IP.

### Leaked Secrets

If a secret ever reaches a remote, rotate it immediately — deleting the line or rewriting history is not enough. Keep secrets in env vars or a vault, and add `.env`, `*.pem`, `*.key`, and local override files to `.gitignore`.

### Dependencies and Supply Chain

- Run the dependency audit before release (`mvn org.owasp:dependency-check:check` / Gradle `dependencyCheckAnalyze`). A critical/high finding reachable in the runtime path blocks the release; unreachable or dev-only findings are tracked for the next cycle.
- Upgrade one dependency per change and read its changelog and the BOM/lockfile diff — not just the version number. A bulk bump that breaks the build hides which package did it.
- Audits only catch known advisories. Before adding a new dependency, check whether the stack already solves it, and review ownership, maintenance, and release age — a plausible-looking group ID can be a typosquat.

### LLM Features

If the service calls an LLM, map the surface to the [OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/):

- Treat model output as untrusted input — never pass it straight into a query, shell command, or file path.
- Assume prompts can be hijacked by untrusted text in context; enforce permissions in code, not in the system prompt.
- Keep secrets and other tenants' data out of prompts; anything in context can be echoed back.
- Give tools/agents minimum permissions, confirm destructive actions, and cap tokens, rate, and recursion depth.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "This is an internal tool, security doesn't matter" | Internal tools get compromised. Attackers target the weakest link. |
| "The framework handles security" | Frameworks provide tools, not guarantees. An ownership check you didn't write isn't there. |
| "The audit passed, so the dependency is safe" | Audits match known advisories; they don't catch a newly malicious package. |
| "It's just a version bump" | A bump is a behavior change you didn't write. Read the changelog. |

## Red Flags

- User input or LLM output passed into a query, shell command, or file path
- Endpoints that check authentication but not ownership/role
- Secrets in source code or commit history
- Server fetches a user-supplied URL without an allowlist
- Wildcard CORS origins, or no rate limiting on auth endpoints
- Bulk dependency bump with no changelog review

## Verification

- [ ] Trust boundaries for the feature are named
- [ ] All external input validated at the boundary; all queries parameterized
- [ ] Authorization (not just authentication) checked on every protected endpoint
- [ ] No secrets in source or git history; error responses expose no internals
- [ ] Dependency audit has no unmitigated reachable critical/high findings
- [ ] Server-side URL fetches and LLM output validated before use (if present)
