---
name: speckit-implement
description: Implement a feature by executing its small task list.
disable-model-invocation: true
---

# Implement a feature

Implement the tasks in order with the repository-local workflow. It needs no CLI installation or `.specify/` directory.

## Input

The user must provide the feature directory, for example:

```text
/speckit-implement specs/photo-organizer
```

If no directory is provided, find directories under `specs/` that contain `spec.md`, `plan.md`, and `tasks.md`. Continue only when exactly one directory matches. Otherwise ask the user to provide the path.

## Steps

1. Read `<FEATURE_DIR>/spec.md`, `<FEATURE_DIR>/plan.md`, and `<FEATURE_DIR>/tasks.md`.
2. Read the relevant source files and project instructions before changing code.
3. Check task dependencies and start with the first incomplete task. Do not skip a failed prerequisite.
4. Implement one task at a time. Follow the plan and existing project patterns.
5. Run the smallest useful validation after each related group of changes. Run the full validation from `tasks.md` before completion.
6. Mark a task `[x]` in `<FEATURE_DIR>/tasks.md` only after its work and validation pass.
7. If a task is unclear or the plan is wrong, stop. Explain the issue and update the plan or task list only with the user's approval.
8. Review the final changes against `spec.md`. Confirm that no requirement is missing and that no unrelated change was added.
9. Report completed tasks, remaining tasks, validation commands, and any failures.

## Done when

- All tasks in `<FEATURE_DIR>/tasks.md` are complete and marked `[x]`.
- The feature matches `spec.md` and `plan.md`.
- The listed validation commands pass, or failures are clearly reported.
