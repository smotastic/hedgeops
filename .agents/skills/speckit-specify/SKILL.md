---
name: speckit-specify
description: Create a small feature specification in a specs/<feature>/ directory.
disable-model-invocation: true
---

# Specify a feature

Create one feature specification with the repository-local workflow. It needs no CLI installation or `.specify/` directory.

## Input

The user message is the feature description. The user may provide `slug=<name>`.

## Steps

1. Read the repository's `README.md`, `AGENTS.md`, and relevant source files. Do not read the whole repository without a reason.
2. Create a short kebab-case slug from the feature description. Use `slug=<name>` when provided.
3. Set `FEATURE_DIR` to `specs/<slug>`. If that directory already exists, stop and ask the user for a different slug.
4. Create `FEATURE_DIR/spec.md` with this structure:

   ```markdown
   # Feature: <name>

   ## Summary
   <What the feature does and why it matters.>

   ## User stories
   ### US1 — <title> (P1)
   As a <user>, I want <action>, so that <benefit>.

   **Acceptance criteria**
   - Given <context>, when <action>, then <result>.

   ## Requirements
   - FR-001: The system shall <testable requirement>.

   ## Edge cases
   - <case and expected behavior>

   ## Out of scope
   - <explicit exclusion>

   ## Assumptions
   - <reasonable assumption>
   ```

5. Write requirements that a test can verify. Keep the specification focused on user value and behavior. Put technical choices in `plan.md`, not `spec.md`.
6. Use reasonable assumptions. Add `[NEEDS CLARIFICATION: ...]` only when the choice blocks the scope or user behavior. Use no more than three markers.
7. Check that every user story has acceptance criteria, every requirement is testable, and the scope is clear.
8. Report `FEATURE_DIR` and `FEATURE_DIR/spec.md`.

## Done when

- `specs/<slug>/spec.md` exists.
- The specification has user stories, testable requirements, acceptance criteria, edge cases, scope, and assumptions.
- The report names the feature directory and spec file.
