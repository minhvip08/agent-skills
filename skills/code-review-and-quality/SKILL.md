---
name: code-review-and-quality
description: Conducts multi-axis code review — correctness, readability, architecture, security, performance. Use before merging any change, whether written by yourself, another agent, or a human.
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

1. **Correctness** — matches the spec/task? Edge cases and error paths handled? Do the tests test the right things?
2. **Readability** — descriptive names, straightforward control flow, no dead code or backwards-compat shims. Are abstractions earning their complexity (don't generalize before the third use case)?
3. **Architecture** — follows existing patterns or justifies a new one; no circular dependencies; feature-specific logic lives in its owning module. A refactor should *remove* complexity, not relocate it — prefer the version where whole branches disappear.
4. **Security** — input validated, queries parameterized, authorization checked, secrets out of code and logs, dependency changes reviewed. See `security-and-hardening`.
5. **Performance** — N+1 queries, unbounded fetches, missing pagination, blocking work on a hot path. See `performance-optimization`.

## Change Sizing

~100 changed lines is comfortably reviewable; ~300 is acceptable as one logical change; ~1000+ is too large — split it (by dependency stack, by file group, or by feature slice). A small diff that pushes an already-large file bigger is a signal to decompose first.

## Review Process

1. **Understand intent** — what is this trying to accomplish, against what spec/task?
2. **Review tests first** — do they exist, cover edge cases, and test behavior rather than implementation? Answer "would they catch a regression?" by experiment, not by reading: invert one condition the change adds (drop a negation, swap `&&` for `||`), run the suite, then restore the file. A mutation that stays green is a finding.
3. **Walk the implementation** against the five axes.
4. **Label every finding by severity:**

   | Prefix | Meaning |
   |--------|---------|
   | **Critical:** | Blocks merge — security, data loss, broken functionality |
   | *(none)* | Required — must address before merge |
   | **Consider:** | Suggestion, not required |
   | **Nit:** | Optional — style, formatting |

   Lead with what matters. One structural problem beats ten nits; don't bury it.
5. **Check the verification story** — what tests ran, did the build pass, was it verified manually.

**Dead code:** anything the diff itself orphaned (an import, variable, or method it made unused) is Required. Pre-existing dead code — mention it and ask; don't demand an unrelated cleanup in this change.

For extra scrutiny, have a fresh-context instance or second model review before a human makes the final call.

## Honesty in Review

Don't rubber-stamp and don't soften real issues. Quantify when you can ("this N+1 adds ~50ms per row" beats "this could be slow"). Push back on approaches with clear problems; if the author has full context and disagrees, defer — but don't accept "I'll clean it up later."

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "The tests pass, so it's good" | Tests don't catch architecture, security, or readability problems — and may not catch the regression either. Mutate to check. |
| "I wrote it, so it's correct" | Authors are blind to their own assumptions. |
| "We'll clean it up later" | Later never comes. Require cleanup before merge. |
| "The refactor makes it cleaner" | Relocating complexity isn't reducing it. |

## Red Flags

- "LGTM" with no evidence of review, or a review that only checks that tests pass
- Large changes that are "too big to review properly" (split them)
- Bug fixes with no regression test
- Findings without severity labels
- A refactor that moves code around without reducing the concepts a reader must hold

## Verification

- [ ] All Critical issues resolved
- [ ] All Required changes resolved or explicitly deferred with justification
- [ ] Tests pass, build succeeds, and the verification story is documented
