---
name: speckit-tasks
description: Create a small dependency-ordered task list from a feature specification and plan.
disable-model-invocation: true
---

# Create implementation tasks

Create an executable task list with the repository-local workflow. It needs no CLI installation or `.specify/` directory.

## Input

The user must provide the feature directory, for example:

```text
/speckit-tasks specs/photo-organizer
```

If no directory is provided, find directories under `specs/` that contain both `spec.md` and `plan.md`. Continue only when exactly one directory matches. Otherwise ask the user to provide the path.

## Steps

1. Read `<FEATURE_DIR>/spec.md` and `<FEATURE_DIR>/plan.md`.
2. Read the files named by the plan. Confirm that task paths match the repository.
3. Create or replace `<FEATURE_DIR>/tasks.md` with this structure:

   ```markdown
   # Tasks: <feature>

   ## Dependencies
   - T001 before T002

   ## Tasks
   - [ ] T001 Inspect and update <path> to support <behavior>.
   - [ ] T002 [P] Add tests for <behavior> in <path>.
   - [ ] T003 Implement <behavior> in <path>.

   ## Validation
   - <command>
   - <expected result>
   ```

4. Use sequential IDs starting at `T001`.
5. Give each task one clear action and at least one exact file path. Use `[P]` only when the task can run in parallel with other incomplete tasks.
6. Order tasks by dependency. Include setup, implementation, tests, integration, and documentation tasks only when the plan needs them.
7. Organize tasks by user story when the specification has multiple stories. Mark story tasks with `[US1]`, `[US2]`, and so on.
8. Include validation commands from the plan. Do not create test tasks unless the project or user asks for tests, but always include the validation needed to prove the feature works.
9. Check that every requirement and plan file has a task.
10. Report the task file path, task count, and validation commands.

## Done when

- `<FEATURE_DIR>/tasks.md` exists.
- Every task has a checkbox, ID, clear action, and file path.
- Tasks are dependency ordered and cover the plan.
