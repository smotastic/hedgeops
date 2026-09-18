---
name: speckit-plan
description: Create a small implementation plan from a feature specification.
disable-model-invocation: true
---

# Plan a feature

Create a practical implementation plan with the repository-local workflow. It needs no CLI installation or `.specify/` directory.

## Input

The user must provide the feature directory, for example:

```text
/speckit-plan specs/photo-organizer
```

If no directory is provided, find directories under `specs/` that contain `spec.md`. Continue only when exactly one directory matches. Otherwise ask the user to provide the path.

## Steps

1. Read `<FEATURE_DIR>/spec.md`.
2. Read `README.md`, `AGENTS.md`, relevant source files, project configuration, and existing tests. Inspect only files needed to understand the change.
3. Identify the existing architecture, test commands, file locations, dependencies, and constraints.
4. Create or replace `<FEATURE_DIR>/plan.md` with this structure:

   ```markdown
   # Implementation plan: <feature>

   ## Goal
   <Short statement of the intended result.>

   ## Technical approach
   <How the existing project will implement the feature.>

   ## Files to change
   | File | Change | Reason |
   | --- | --- | --- |

   ## Data and interfaces
   <Data changes, public interfaces, and compatibility rules. Write “None” when not needed.>

   ## Validation
   - <test or command>
   - <manual check, if needed>

   ## Risks and decisions
   - <risk, decision, or assumption>
   ```

5. Prefer the project's existing patterns. Do not invent frameworks, services, or files when the repository already has a suitable pattern.
6. Keep the plan specific enough for another agent to implement. Name files and test commands. Resolve simple unknowns by inspecting the repository. Mark only blocking unknowns as `[NEEDS CLARIFICATION: ...]`.
7. Check that the plan covers every user story and requirement in `spec.md`.
8. Report the plan path and the validation commands.

## Done when

- `<FEATURE_DIR>/plan.md` exists.
- The plan names the files, technical approach, validation steps, and risks.
- Every requirement in `spec.md` maps to an implementation or validation step.
