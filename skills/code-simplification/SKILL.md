---
name: code-simplification
description: Simplifies code for clarity without changing behavior. Use when code works but is harder to read, maintain, or extend than it should be, or when review flags unnecessary complexity.
---

# Code Simplification

## Overview

Reduce complexity while preserving exact behavior. The goal isn't fewer lines — it's code a new teammate would understand faster than the original.

## When to Use

- Code works but feels heavier than it needs to be
- Code review flags readability or complexity issues
- Deeply nested logic, long methods, or unclear names

**When NOT to use:** code is already clean; you don't yet understand what it does; it's performance-critical and the simpler version would be measurably slower.

## Principles

1. **Preserve behavior exactly** — same outputs, errors, and side effects. If unsure a change preserves behavior, don't make it.
2. **Follow project conventions** — simplification that breaks consistency is churn.
3. **Clarity over cleverness** — explicit beats compact when the compact form needs a mental pause to parse.
4. **Don't over-simplify** — inlining a helper that named a concept, merging two simple methods into one complex one, or removing an abstraction that exists for testability are regressions dressed as improvements.
5. **Stay in scope** — simplify recently touched code only, as its own change (see `incremental-implementation` for scope rules).

## Process

1. **Understand before touching (Chesterton's Fence).** What is this code's responsibility, what calls it, what edge cases does it handle, why was it written this way? Check `git blame`. If you can't answer, read more first.
2. **Identify opportunities:**

   | Pattern | Simplification |
   |---------|----------------|
   | Deep nesting (3+ levels) | Guard clauses / early returns |
   | Long methods (50+ lines), multiple responsibilities | Split into focused methods |
   | Boolean parameter flags | Overloads, a builder, or an options object |
   | Generic names (`data`, `temp`, `result`) | Rename to describe content |
   | Duplicated logic (5+ lines repeated) | Extract to a shared method |
   | Dead code, commented-out blocks | Remove after confirming it's unused |
   | Wrapper that adds no value | Inline it |
   | Comment explaining *what* | Delete — the code should say it |
   | Comment explaining *why* | Keep — intent the code can't express |

3. **Apply one simplification at a time; run tests after each.** If tests fail, revert and reconsider. For refactors touching 500+ lines, prefer automated tooling (IDE refactorings, OpenRewrite) over hand edits.
4. **Judge the whole:** is it genuinely easier to understand, and would a teammate approve the diff? If not, revert — not every attempt succeeds.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "Fewer lines is always simpler" | A dense one-liner isn't simpler than a clear if/else. Simplicity is comprehension speed, not line count. |
| "This abstraction might be useful later" | If it's not used now, it's complexity without value — remove it, re-add when needed. |
| "I had to tweak the test, but it's the same behavior" | A test that must change to pass is evidence behavior changed. |

## Red Flags

- Simplification requires modifying tests to pass
- "Simplified" code is longer or harder to follow than the original
- Removing error handling to "make it cleaner"
- Simplifying code you don't fully understand
- Many simplifications batched into one large commit

## Verification

- [ ] All existing tests pass without modification
- [ ] Build succeeds with no new warnings
- [ ] Each simplification is a separate, reviewable change
- [ ] No error handling removed or weakened
