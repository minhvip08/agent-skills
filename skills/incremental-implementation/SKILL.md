---
name: incremental-implementation
description: Delivers changes incrementally. Use when implementing any feature or change that touches more than one file. Use when you're about to write a large amount of code at once, or when a task feels too big to land in one step.
---

# Incremental Implementation

## Overview

Build in thin vertical slices — implement one piece, test it, verify it, then expand. Each increment should leave the system in a working, testable state.

## When to Use

- Implementing any multi-file or multi-layer change
- Building a new feature from a task breakdown
- Any time you're tempted to write more than ~100 lines before testing

**When NOT to use:** single-file, single-method changes where the scope is already minimal.

## The Increment Cycle

For each slice: **implement** the smallest complete piece → **test** (run the suite, or write one if missing) → **verify** (tests pass, build succeeds) → **stage** only the files this slice touched (`git add <files>`, never `git add -A`) → move to the next slice, carrying forward.

Don't commit after every slice. Staging after each verified increment keeps the working tree organized and lets you review exactly what each slice changed, but committing is a separate decision — made once, at a natural checkpoint, or when the task is done — not an automatic step in the per-slice loop.

## Slicing Strategies

- **Vertical (preferred)** — one complete path per slice: e.g. "create a task" (entity + repository + service + endpoint), then "list tasks," then "update," then "delete." Each slice is a working, testable path end to end.
- **Contract-first** — when a consumer depends on your API: define the contract (DTOs/interface) first, implement the service against it with tests, let the consumer build against the contract in parallel, then integrate.
- **Risk-first** — build the riskiest or most uncertain piece first (e.g. prove a message-queue connection works) so a failure surfaces before you've built on top of it.

## Implementation Rules

1. **Simplicity first.** Before coding, ask "what's the simplest thing that could work?" Three similar lines beat a premature abstraction — implement the naive, obviously-correct version, optimize only after tests prove correctness. Don't add configurability or flexibility nobody asked for, and don't write error handling for scenarios that can't actually happen — both are complexity paid for now against a benefit that may never arrive.
2. **Scope discipline.** Touch only what the task requires — don't clean up adjacent code, modernize syntax you're only reading, or add features not in the spec "because they seem useful." If you notice something worth fixing outside scope, note it and ask, don't fix it inline. Exception: if your own change leaves behind an unused import, variable, or method, remove it — that's cleanup of your own mess, not scope creep. Pre-existing dead code you didn't just orphan is out of scope; note it, don't touch it. This applies to tooling too: run the formatter, linter, or build only against the files your change touched (e.g. `git diff --name-only`), never the whole repo — a full-tree format/lint pass reformats files nobody asked you to touch and buries the real diff in noise.
3. **One thing at a time.** Each increment changes one logical thing — don't mix a new feature with an unrelated refactor or config change in the same staged batch.
4. **Keep it compilable.** After each increment, the project must build and existing tests must pass — never leave the codebase broken between slices.
5. **Feature flags for incomplete work.** Gate an unfinished feature behind a flag checked at the entry point so you can still stage small increments.
6. **Safe defaults.** New options are opt-in and disabled by default, not opt-out.
7. **Rollback-friendly.** Prefer additive changes; keep modifications to existing code minimal and focused; pair a migration with its rollback; don't delete something and replace it within the same increment.

When directing an agent, be explicit about what's in scope for the increment (e.g., "just the entity and repository — don't touch the controller yet") and name the verification commands to run afterward.

## Increment Checklist

- [ ] The change does one thing and does it completely
- [ ] All existing tests still pass (`mvn test` / `./gradlew test`)
- [ ] The build succeeds (`mvn verify` / `./gradlew build`)
- [ ] The new functionality works as expected
- [ ] The change is staged (`git add`), scoped to only the files this increment touched — not committed yet

Run each command after a change that could affect it — don't re-run on unchanged code as mere reassurance.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'll test it all at the end" | A bug in slice 1 makes every slice built on it wrong. Test each slice. |
| "It's faster to do it all at once" | Feels faster until something breaks and you can't tell which of 500 changed lines caused it. |
| "I'll just `git add -A` at the end" | Stage each increment as you verify it — scoped to the files it touched. A blanket add at the end re-mixes everything the slicing was meant to separate. |
| "This refactor is small enough to include" | Refactors mixed with features are harder to review and debug. Separate them. |

## Red Flags

- More than ~100 lines written without running tests
- Multiple unrelated changes in a single increment
- Build or tests broken between increments
- Building abstractions before a third use case demands them
- Touching files outside the task scope "while I'm here"
- Running a formatter, linter, or build across the whole repo instead of just the changed files

## Verification

- [ ] Each increment was individually tested and staged (scoped to the files it touched)
- [ ] The full test suite passes and the build is clean
- [ ] The feature works end-to-end as specified
- [ ] No unstaged changes remain; committing is a separate, deliberate step once the task is reviewed
