# Hedgeops

A smart-home system.

## Included files

- `AGENTS.md` contains short, project-wide agent rules.
- `docs/engineering/` contains reusable engineering guidance.
- `docs/adr/` is the default location for project ADRs.
- `.agents/skills/` contains user-invoked skills for the small spec workflow, ADRs, commits, pushes, and pull requests.
- `.github/pull_request_template.md` provides the pull request form.
- `.github/workflows/validate.yml` validates repository changes.

## Small spec workflow

This repository uses a local, lightweight workflow. It does not require the official Spec Kit CLI or a `.specify/` directory.

Each feature stores its artifacts in one directory:

```text
specs/<feature>/
├── spec.md
├── plan.md
└── tasks.md
```

Run the skills in order. Pass the feature directory to every step after `specify`:

```text
/speckit-specify Add a photo organizer with albums grouped by date.
/speckit-plan specs/photo-organizer
/speckit-tasks specs/photo-organizer
/speckit-implement specs/photo-organizer
```

The skills are intentionally small. They do not create branches, run hooks, install tools, or create extra design documents.

## Progressive disclosure

`AGENTS.md` is intentionally short. It points to detailed guidance instead of loading every document for every task. Read a linked document only when its task condition applies.

The main references are:

- [AGENTS.md guidelines](docs/engineering/agents-md-guidelines.md)
- [ADR template](docs/engineering/adr-template.md)
- [`write-adr` skill](.agents/skills/write-adr/SKILL.md)
- [`commit-and-push` skill](.agents/skills/commit-and-push/SKILL.md)
- [`create-pr` skill](.agents/skills/create-pr/SKILL.md)

## Skills

These skills are user-invoked. They do not run Git or GitHub changes without an explicit request.

- `write-adr` creates, updates, supersedes, or reviews ADRs.
- `commit-and-push` stages, commits, and pushes all current changes. Its commit format matches the `yeet` Pi extension.
- `create-pr` uses the GitHub CLI (`gh`) to create a regular open pull request.
