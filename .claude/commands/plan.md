---
description: Break work into small verifiable tasks with acceptance criteria and dependency ordering
---

Invoke the agent-skills:planning-and-task-breakdown skill.

Find the task's folder under `tasks/` (the most recently modified `tasks/<date>-<task-name>/` containing a `SPEC.md`; if more than one is a plausible match, ask the user which task this is for). Read that spec and the relevant codebase sections. Then:

1. Enter plan mode — read only, no code changes
2. Identify the dependency graph between components
3. Slice work vertically (one complete path per task, not horizontal layers)
4. Write tasks with acceptance criteria and verification steps
5. Add checkpoints between phases
6. Present the plan for human review

Save the plan and task list into the same task folder: `tasks/<date>-<task-name>/plan.md` and `.../todo.md`. If no spec exists yet, create the folder now using today's date and a kebab-case slug of the task name.
