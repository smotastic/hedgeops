# Architecture Decision Record template

An Architecture Decision Record (ADR) records one important technical decision and why the project made it.

## File name

Store ADRs in `docs/adr/` with this format:

```text
0001-short-kebab-case-title.md
```

Use a four-digit number. Start at `0001` and increase the number by one. Keep the title short and use lowercase kebab-case, which means lowercase words separated by hyphens.

## Template

Copy this content into a new ADR. Replace every placeholder.

```markdown
---
number: "0001"
title: "Use [technology or pattern]"
status: "Proposed"
date: "YYYY-MM-DD"
supersedes: null
superseded_by: null
---

# ADR-0001: Use [technology or pattern]

## Context

Describe the problem, constraints, and forces that led to this decision.

## Decision

State the decision in clear, direct language.

## Consequences

Describe the expected results of the decision.

### Positive

- [Positive consequence]

### Negative

- [Negative consequence or trade-off]

## Alternatives considered

Describe the important alternatives and why they were not selected.

## Links

- [Optional supporting link]
```

## Required content

Every ADR must include YAML frontmatter, a metadata block at the top of the Markdown file, with:

- `number`
- `title`
- `status`
- `date`
- Context
- Decision
- Consequences
- Alternatives considered

The Links section is optional. Use `supersedes` and `superseded_by` when one ADR replaces another.

## Status values

Use one of these values:

- `Proposed`: under discussion or awaiting approval.
- `Accepted`: approved and used by the project.
- `Rejected`: considered and not selected.
- `Deprecated`: no longer recommended, but not replaced by a later ADR.
- `Superseded`: replaced by another ADR.

Do not mark an ADR as `Accepted` without the user's approval.
