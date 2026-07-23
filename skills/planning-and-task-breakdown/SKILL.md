---
name: planning-and-task-breakdown
description: Breaks work into ordered tasks. Use when you have a spec or clear requirements and need to break work into implementable tasks. Use when a task feels too large to start, when you need to estimate scope, or when parallel work is possible.
---

# Planning and Task Breakdown

## Overview

Decompose work into small, verifiable tasks with explicit acceptance criteria. Every task should be small enough to implement, test, and verify in a single focused session.

## When to Use

- You have a spec and need to break it into implementable units
- A task feels too large or vague to start
- Work needs to be parallelized across multiple agents or sessions

**When NOT to use:** single-file changes with obvious scope, or the spec already contains well-defined tasks.

## The Planning Process

1. **Enter plan mode.** Read the spec and relevant code, identify existing patterns, map dependencies, note risks — read-only, no code written. Output is a plan saved to `tasks/plan.md` and a checklist to `tasks/todo.md`.
2. **Map the dependency graph.** For a typical backend feature: `entity/schema → repository → service → controller/endpoint → validation`. Migrations and seed data sit alongside the schema. Order implementation bottom-up.
3. **Slice vertically, not horizontally.** Don't do "all entities, then all services, then all controllers" — instead, "user can register" (entity + repository + service + endpoint), then "user can log in," then the next feature slice. Each slice is working, testable functionality end to end.
4. **Write each task with:**
   - **Description** — one paragraph
   - **Acceptance criteria** — specific, testable conditions
   - **Verification** — test command (`mvn test -Dtest=FeatureNameTest` / `./gradlew test --tests FeatureName*`), build command, manual check if needed
   - **Dependencies** — task numbers, or "None"
   - **Files likely touched**
5. **Order and checkpoint.** Satisfy dependencies first, put high-risk tasks early (fail fast), and add a checkpoint every 2–3 tasks (tests pass, build clean, core flow works, review before continuing).

## Task Sizing

| Size | Files | Example |
|------|-------|---------|
| **S** | 1–2 | Add a new endpoint |
| **M** | 3–5 | One feature slice (e.g. registration flow) |
| **L** | 5–8 | Multi-component feature — consider splitting |
| **XL** | 8+ | **Too large — break it down** |

Break a task down further if: it would take more than one focused session, you can't state acceptance criteria in ≤3 bullets, it spans two independent subsystems (e.g. auth and billing), or the title has an "and" in it.

## Plan Document Template

```markdown
# Implementation Plan: [Feature Name]

## Overview
[One paragraph]

## Architecture Decisions
- [Decision and rationale]

## Task List
### Phase 1: Foundation
- [ ] Task 1: ...
### Checkpoint
- [ ] Tests pass, build clean

### Phase 2: Core Features
- [ ] Task 2: ...
### Checkpoint
- [ ] End-to-end flow works

## Risks and Mitigations
| Risk | Impact | Mitigation |
|------|--------|------------|

## Open Questions
- [Needs human input]
```

Save to `tasks/plan.md` (plan) and `tasks/todo.md` (checklist) — the convention expected by `/build` and other downstream tooling.

## Parallelization

Safe to parallelize: independent feature slices, tests for already-implemented features, documentation. Must be sequential: database migrations, shared state changes, dependency chains. Needs coordination: features sharing an API contract — define the contract first, then parallelize.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'll figure it out as I go" | That's how you end up with a tangled mess. Minutes of planning saves hours. |
| "The tasks are obvious" | Write them down anyway — it surfaces hidden dependencies and edge cases. |
| "Planning is overhead" | Planning is the task. Implementation without a plan is just typing. |

## Red Flags

- Starting implementation without a written task list
- Tasks that say "implement the feature" with no acceptance criteria
- All tasks are XL-sized, or no checkpoints exist between phases
- Dependency order isn't considered

## Verification

- [ ] Every task has acceptance criteria and a verification step
- [ ] Task dependencies are identified and ordered correctly
- [ ] Checkpoints exist between major phases
- [ ] The human has reviewed and approved the plan
