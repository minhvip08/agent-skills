---
name: code-review-and-quality
description: Conducts multi-axis code review. Use before merging any change. Use when reviewing code written by yourself, another agent, or a human. Use when you need to assess code quality across multiple dimensions before it enters the main branch.
---

# Code Review and Quality

## Overview

Every change gets reviewed before merge across five axes: correctness, readability, architecture, security, performance.

**Approval standard:** approve when the change definitely improves overall code health, even if imperfect. Don't block a change just because it isn't how you'd have written it.

## When to Use

- Before merging any change
- After completing a feature or bug fix (review the fix and its regression test)
- When another agent or model produced code you need to evaluate

## The Five Axes

1. **Correctness** — matches the spec/task? Edge cases and error paths handled? Do the tests actually test the right things?
2. **Readability** — descriptive names, straightforward control flow, no dead code or backwards-compat shims. Could this be fewer lines without losing clarity? Are abstractions earning their complexity (don't generalize before the third use case)?
3. **Architecture** — follows existing patterns or justifies a new one; no circular dependencies; a refactor should *remove* complexity, not relocate it — prefer the version where whole branches disappear over one that just re-centralizes the same logic. Feature-specific logic belongs in its owning module, not a shared one.
4. **Security** — see `security-and-hardening`. Input validated? Secrets out of code/logs? Queries parameterized? Auth checked?
5. **Performance** — see `performance-optimization`. N+1 queries? Unbounded fetches? Missing pagination? Synchronous work that should be async?

## Change Sizing

~100 changed lines is comfortably reviewable; ~300 is acceptable as one logical change; ~1000+ is too large — split it (by dependency stack, by file group, or vertically by feature slice). Watch total file size too: a small diff that pushes an already-large file bigger is a signal to decompose first. Separate refactoring from feature work — they're two changes.

## Review Process

1. **Understand intent** — what is this trying to accomplish, against what spec/task?
2. **Review tests first** — do they exist, cover edge cases, and test behavior rather than implementation?
3. **Walk the implementation** against the five axes above.
4. **Categorize findings by severity** so the author knows what's required:

   | Prefix | Meaning |
   |--------|---------|
   | *(none)* | Required — must address before merge |
   | **Critical:** | Blocks merge — security, data loss, broken functionality |
   | **Nit:** | Optional — style, formatting |
   | **Consider:** | Suggestion, not required |

   Lead with what matters — correctness and security first. One structural problem beats ten nits; don't bury it.
5. **Check the verification story** — what tests ran, did the build pass, was it verified manually.

For extra scrutiny, have a second model (or fresh-context instance) review before a human makes the final call — different reviewers catch different blind spots.

## Honesty in Review

Don't rubber-stamp ("LGTM" with no evidence of review) and don't soften real issues. Quantify problems when you can ("this N+1 adds ~50ms per row" beats "this could be slow"). Push back on approaches with clear problems instead of deferring to the author. If the author has full context and disagrees, defer to their judgment — but don't accept "I'll clean it up later"; that cleanup rarely happens.

## Dependency Discipline

Before adding a dependency: does the existing stack already solve this? Is it maintained? Any known vulnerabilities? Prefer the standard library over a new dependency. When upgrading an existing one, read the changelog (not just the version number), upgrade one dependency per change so a broken build points at a single cause, and verify with a green test suite before and after.

## Dead Code Hygiene

After a refactor, list anything now unreachable or unused and ask before deleting it — don't leave it lying around, but don't silently remove things you're not sure about either.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "It works, that's good enough" | Unreadable, insecure, or architecturally wrong code creates debt that compounds. |
| "I wrote it, so it's correct" | Authors are blind to their own assumptions. Every change benefits from another set of eyes. |
| "We'll clean it up later" | Later never comes. Require cleanup before merge. |
| "The tests pass, so it's good" | Tests don't catch architecture, security, or readability problems. |
| "The refactor makes it cleaner" | Relocating complexity isn't reducing it — look for the version where branches disappear. |
| "It's just a version bump" | A bump is a behavior change you didn't write. Read the changelog. |

## Red Flags

- Changes merged without any review, or a review that only checks whether tests pass
- "LGTM" with no evidence of actual review
- Security-sensitive changes without security-focused review
- Large changes that are "too big to review properly" (split them)
- Bug fixes with no regression test
- Comments without severity labels
- A refactor that moves code around without reducing the concepts a reader must hold
- A bulk dependency bump with no changelog review

## Verification

- [ ] All Critical issues resolved
- [ ] All Required changes resolved or explicitly deferred with justification
- [ ] Tests pass, build succeeds
- [ ] The verification story is documented
- [ ] Dependency upgrades reviewed against their changelog, one per change
