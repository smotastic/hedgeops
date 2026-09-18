---
name: commit-and-push
description: Stage, commit, and push all current repository changes using the project's Better Commits format.
disable-model-invocation: true
---

# Commit and push

Use this skill only when the user explicitly asks to commit and push the current repository changes. This workflow matches the `yeet` Pi extension.

Text after the skill name is extra context for the commit message. Use it to understand the change. Do not copy it into the commit without checking that it is accurate.

## Preconditions

1. Confirm that the current directory is a Git repository.
2. Read the current status, branch, recent commits, and configured remotes.
3. Check project-defined validation commands in `AGENTS.md`, `README.md`, `package.json`, `Makefile`, and clear project task files.
4. Run only checks that the repository clearly defines. Do not guess commands.
5. If a defined check fails, report the failure and stop. Continue only when the user explicitly approves continuing.

Completion criterion: the repository state, current branch, remotes, and defined checks are known, and every defined check that applies has passed or has explicit approval to bypass the failure.

## Stage and inspect

1. Stage all current changes with `git add -A`.
2. Inspect the complete staged diff with `git diff --cached`.
3. Check the staged diff for secrets, credentials, private keys, tokens, and unrelated files.
4. If the staged diff contains a likely secret or unsafe unrelated change, stop and report it. Do not commit it.

Completion criterion: the staged diff contains the intended current changes and no likely secret or unrelated change.

## Write the commit

Create exactly one commit for the staged changes. Use this format:

```text
<emoji> <type>(<scope>): <subject>

[optional body]

[optional footer]
```

Use one of these types and emojis:

- `✨ feat`: a new feature
- `🐛 fix`: a bug fix
- `📝 docs`: documentation changes only
- `💄 style`: formatting with no logic change
- `♻️ refactor`: code restructuring with no feature or fix
- `⚡️ perf`: a performance improvement
- `✅ test`: adding or updating tests
- `🏗️ build`: build system or dependency changes
- `👷 ci`: CI or delivery configuration changes
- `🔧 chore`: maintenance or tooling
- `⏪️ revert`: reverting a previous commit
- `🔒️ security`: fixing a security issue
- `🚀 deploy`: deployment work

Commit rules:

- Keep the subject to 72 characters or fewer.
- Use imperative mood, such as `add`, not `added`.
- Do not end the subject with a period.
- Use a lowercase scope when a scope helps.
- Use the body to explain why, not what.
- Wrap the body at 72 characters.
- Add issue references in a footer when the user provides them.
- Keep the message concise.

Do not create an empty commit. Do not bypass hooks with `--no-verify`.

Completion criterion: one commit exists with the staged changes and a valid Better Commits message.

## Push

1. Push the current branch. Preserve the current branch; do not create or switch branches.
2. If the branch has no upstream, use `git push --set-upstream origin <branch>`.
3. If no remote is configured, do not push. Report that the commit is local.
4. Do not force-push or rewrite history.
5. Convert SSH GitHub remotes to HTTPS when reporting URLs.
6. Report the commit, branch, push result, and the repository URL. On a non-default branch, also report a GitHub compare URL for opening a pull request against the default branch when it can be resolved.

Completion criterion: the commit is pushed to its upstream branch, or the skill clearly reports why it remains local, and the final URLs are shown when available.
