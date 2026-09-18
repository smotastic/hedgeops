---
name: write-adr
description: Create, update, supersede, or review Architecture Decision Records in docs/adr.
disable-model-invocation: true
---

# Write an ADR

Use this skill only when the user explicitly asks to create, update, supersede, or review an Architecture Decision Record (ADR).

## Source of truth

Before any ADR work, read [`docs/engineering/adr-template.md`](../../../docs/engineering/adr-template.md). Follow its frontmatter fields, status values, numbering, and required sections.

## Choose the action

Identify one action from the user's request:

- **Create**: add a new ADR.
- **Update**: change an existing ADR without changing its number or path.
- **Supersede**: create a replacement ADR and update the old ADR after approval.
- **Review**: inspect an ADR and report problems without changing files.

If the action or target is unclear, ask the user before reading or changing files.

## Create an ADR

1. Create `docs/adr/` if it does not exist.
2. List existing files in `docs/adr/`.
3. Find the highest existing four-digit ADR number. Use the next number. Check that the target path does not exist.
4. Convert the title to lowercase kebab-case for the file name.
5. Ask for missing information. The required decision material is context, decision, consequences, and alternatives considered. Use `Proposed` unless the user supplies another status.
6. Use the current date in `YYYY-MM-DD` format unless the user supplies a different date.
7. Draft the complete file with YAML frontmatter, the metadata block at the top of the Markdown file, and the required sections.
8. Show the proposed path and draft. Wait for approval before writing.
9. Write exactly one new ADR after approval.

Completion criterion: the new ADR exists at the next unused four-digit path, contains valid frontmatter and all required sections, and matches the approved draft.

## Update an ADR

1. Find the requested ADR by number or file name.
2. Read the complete file.
3. Preserve its number and path.
4. Change only the requested content. Keep the required frontmatter and sections.
5. Show the proposed path and changes. Wait for approval before writing.
6. Write the approved update.

Do not change an ADR to `Accepted` without the user's approval.

Completion criterion: the requested ADR contains the approved changes, keeps its original number and path, and still passes the template requirements.

## Supersede an ADR

1. Read the old ADR and the ADR template.
2. Create the new ADR using the next unused number.
3. Add the old ADR number to the new ADR's `supersedes` field.
4. Set the old ADR's `status` to `Superseded` and add the new ADR number to its `superseded_by` field.
5. Show both proposed files and the changes to the old file. Wait for approval before writing.
6. Write the new ADR and update the old ADR only after approval.

Completion criterion: the new ADR names the old ADR, the old ADR names the new ADR, both files contain valid required sections, and the old ADR has status `Superseded`.

## Review an ADR

Review without changing files. Check:

- The file name uses a four-digit number and lowercase kebab-case title.
- YAML frontmatter contains `number`, `title`, `status`, and `date`.
- The number matches the file name and heading.
- The status is one of the allowed values.
- Context, Decision, Consequences, and Alternatives considered are present and useful.
- Supersession fields point to existing ADRs when present.
- Links are clear and usable when present.

Report each issue with its file and section. Do not edit the ADR during review.

Completion criterion: the report checks every rule above and clearly states whether the ADR passes.
