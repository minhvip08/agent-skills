---
name: spec-driven-development
description: Creates specs before coding. Use when starting a new project, feature, or significant change and no specification exists yet. Use when requirements are unclear, ambiguous, or only exist as a vague idea.
---

# Spec-Driven Development

## Overview

Write a structured specification before writing any code. The spec is the shared source of truth between you and the human engineer — what we're building, why, and how we'll know it's done.

## When to Use

- Starting a new project or feature
- Requirements are ambiguous or incomplete
- The change touches multiple modules or would take more than ~30 minutes to implement

**When NOT to use:** single-line fixes, typo corrections, or changes where requirements are unambiguous and self-contained.

## Output Location

Every generated artifact for a task lives together in one folder: `tasks/<YYYY-MM-DD>-<kebab-case-task-name>/`. The spec is `SPEC.md` inside that folder (not the repo root) — this keeps one task's spec, plan, and todo list together and stops the next task from overwriting the previous one.

## The Gated Workflow

```
SPECIFY ──→ PLAN ──→ TASKS ──→ IMPLEMENT
   │          │        │          │
   ▼          ▼        ▼          ▼
 Human      Human    Human      Human
 reviews    reviews  reviews    reviews
```

Do not advance to the next phase until the current one is validated.

### Phase 1: Specify

Ask clarifying questions until requirements are concrete, and surface your assumptions before writing anything:

```
ASSUMPTIONS I'M MAKING:
1. This is a Spring Boot service (based on existing pom.xml/build.gradle)
2. Auth uses session-based cookies, not JWT
3. The database is PostgreSQL
→ Correct me now or I'll proceed with these.
```

Cover these six areas:

1. **Objective** — what we're building, why, who the user is, what success looks like
2. **Commands** — full executable commands (`mvn test`, `mvn verify`, `./gradlew build`, etc.), not just tool names
3. **Project structure** — where source, tests, and docs live (e.g. Maven/Gradle standard layout: `src/main/java`, `src/test/java`)
4. **Code style** — one real code snippet beats three paragraphs of description
5. **Testing strategy** — framework (JUnit 5, Kotest), coverage expectations, which test levels cover which concerns
6. **Boundaries** — three tiers: always do (run tests before commit, validate input), ask first (schema changes, new dependencies, CI config), never do (commit secrets, remove failing tests without approval)

**Spec template:**

```markdown
# Spec: [Feature Name]

## Objective
[What and why; success criteria]

## Tech Stack
[Language/framework/key dependencies with versions]

## Commands
[Build, test, lint — full commands]

## Project Structure
[Directory layout]

## Code Style
[Example snippet + conventions]

## Testing Strategy
[Framework, test locations, coverage requirements]

## Boundaries
- Always: [...]
- Ask first: [...]
- Never: [...]

## Success Criteria
[Specific, testable conditions]

## Open Questions
[Anything needing human input]
```

Reframe vague requirements as testable success criteria: "make it faster" becomes "p95 API latency < 200ms" — something you can loop toward instead of guess at.

### Phase 2: Plan

With the validated spec, identify components and their dependencies, implementation order, risks, and what can be parallelized. Follow `planning-and-task-breakdown` for the mechanics — it's the canonical source. Save the plan and task list into the same task folder as the spec: `tasks/<YYYY-MM-DD>-<task-name>/plan.md` and `.../todo.md`.

### Phase 3: Tasks

Break the plan into tasks completable in a single focused session, each with acceptance criteria, a verification step, and dependency ordering. Follow `planning-and-task-breakdown` for sizing and ordering.

### Phase 4: Implement

Execute tasks one at a time following `incremental-implementation`. Use `context-engineering` to load the right spec sections and source files at each step instead of flooding the agent with the entire spec.

## Keeping the Spec Alive

Update the spec when decisions or scope change — before implementing the change, not after. Commit the spec alongside the code, and reference the relevant spec section in each change description.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "This is simple, I don't need a spec" | Simple tasks still need acceptance criteria — a two-line spec is fine. |
| "I'll write the spec after I code it" | That's documentation, not specification. The value is in forcing clarity *before* code. |
| "The spec will slow us down" | A 15-minute spec prevents hours of rework. |
| "Requirements will change anyway" | That's why the spec is a living document — an outdated spec still beats no spec. |

## Red Flags

- Starting to write code without any written requirements
- Asking "should I just start building?" before clarifying what "done" means
- Implementing features not mentioned in any spec or task list
- Making architectural decisions without documenting them

## Verification

- [ ] The spec covers all six core areas
- [ ] The human has reviewed and approved the spec
- [ ] Success criteria are specific and testable
- [ ] Boundaries (Always/Ask First/Never) are defined
- [ ] The spec is saved to `tasks/<date>-<task-name>/SPEC.md`
