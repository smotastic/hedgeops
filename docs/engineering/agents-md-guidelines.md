# AGENTS.md guidelines

`AGENTS.md` gives coding agents project instructions. The root file is loaded for every task, so keep it short and focused.

## Root file rules

Keep only information that applies to every task:

- One sentence that describes the project.
- The package manager when it is not npm.
- Build and type-check commands when they are not clear from the project files.
- Rules that are required for every change.
- Pointers to detailed guidance.

Do not put every project fact in the root file. Do not list a large directory map. File paths change, and stale paths can mislead the agent.

## Progressive disclosure

Put task-specific guidance in a separate document. Link it from `AGENTS.md` with the task that requires it.

Good pointer:

```markdown
When changing database migrations, read the migration guide in `docs/database-migrations.md`.
```

Weak pointer:

```markdown
See the docs for more information.
```

A good pointer tells the agent what the document covers and when to read it. Do not tell the agent to read every document before every task.

## Where guidance belongs

Use these locations:

- Root `AGENTS.md`: rules for every task in the repository.
- A nested `AGENTS.md`: rules for one directory or package.
- A linked document: detailed guidance for one domain, tool, or workflow.
- A skill: a repeatable workflow that the agent should follow when requested.

Keep one rule in one authoritative place. Link to it instead of copying it into several files.

## Writing rules

Write instructions that change agent behavior. Use direct language and concrete conditions.

Prefer:

```markdown
When writing TypeScript, read `docs/typescript.md`.
```

Avoid:

```markdown
Write clean and maintainable code.
```

Keep stable concepts and decisions. Avoid details that become stale, such as exact file paths that the agent can discover from the repository.

## Change checklist

Before adding a rule to root `AGENTS.md`, check:

- Does it apply to every task?
- Is it specific enough to change behavior?
- Can the agent discover it from the project files instead?
- Does a linked document or skill already own this rule?
- Does the link state when the agent should read the target?

If a rule fails these checks, move it to a focused document or remove it.
