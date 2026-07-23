---
name: code-simplification
description: Simplifies code for clarity. Use when refactoring code for clarity without changing behavior. Use when code works but is harder to read, maintain, or extend than it should be. Use when reviewing code that has accumulated unnecessary complexity.
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

1. **Preserve behavior exactly** — same inputs/outputs, same error behavior, same side effects. If unsure a change preserves behavior, don't make it.
2. **Follow project conventions** — match existing naming, error handling, and style. Simplification that breaks consistency is churn, not simplification.
3. **Clarity over cleverness** — explicit beats compact when the compact form needs a mental pause to parse.
4. **Don't over-simplify** — inlining a helper that named a concept, merging two simple methods into one complex one, or removing an abstraction that exists for testability are all regressions dressed as improvements.
5. **Scope to what changed** — simplify recently touched code; don't drive-by refactor unrelated code.

## Process

1. **Understand before touching (Chesterton's Fence).** What is this code's responsibility, what calls it, what edge cases does it handle, and why might it have been written this way? Check `git blame` for context. If you can't answer these, read more before simplifying.
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

3. **Apply one simplification at a time; run tests after each.** If tests fail, revert and reconsider. Submit refactors separately from feature/bug-fix changes. If a refactor would touch 500+ lines, prefer automation (codemods) over hand edits.
4. **Verify the whole:** is it genuinely easier to understand, is the diff clean, would a teammate approve it? If not, revert — not every attempt succeeds.

## Examples

```java
// Nested null checks -> early return
public Result process(Data data) {
    if (data == null) throw new NullPointerException("Data is null");
    if (!data.isValid()) throw new IllegalArgumentException("Invalid data");
    return doWork(data);
}

// Manual loop -> stream
List<User> activeUsers = users.stream()
    .filter(User::isActive)
    .collect(Collectors.toList());
```

```kotlin
// Verbose null check -> elvis operator
val displayName = user.nickname ?: user.fullName

// Manual loop -> filter
val activeUsers = users.filter { it.isActive }

// Redundant boolean return -> expression body
fun isValid(input: String): Boolean = input.length in 1..99
```

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "It's working, no need to touch it" | Hard-to-read working code is hard to fix when it breaks. |
| "Fewer lines is always simpler" | A dense one-liner isn't simpler than a clear if/else. Simplicity is comprehension speed, not line count. |
| "I'll simplify this unrelated code too while I'm here" | Unscoped changes create noisy diffs and regression risk. Stay focused. |
| "This abstraction might be useful later" | If it's not used now, it's complexity without value — remove it, re-add when needed. |
| "I'll refactor while adding this feature" | Mixed changes are harder to review and revert. Separate them. |

## Red Flags

- Simplification requires modifying tests to pass (behavior likely changed)
- "Simplified" code is longer or harder to follow than the original
- Removing error handling to "make it cleaner"
- Simplifying code you don't fully understand
- Batching many simplifications into one large, hard-to-review commit

## Verification

- [ ] All existing tests pass without modification
- [ ] Build succeeds with no new warnings
- [ ] Each simplification is a separate, reviewable change
- [ ] No error handling removed or weakened
- [ ] No dead code left behind
