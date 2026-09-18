---
name: create-pr
description: Push the current feature branch and create a regular open GitHub pull request with gh.
disable-model-invocation: true
---

# Create a pull request

Use this skill only when the user explicitly asks to create a GitHub pull request. Use the GitHub CLI (`gh`) for GitHub operations.

Text after the skill name is context for the pull request title and body. Use it to describe the change. Do not invent issue numbers, reviewers, labels, or test results.

## Resolve the repository

1. Confirm that the current directory is a Git repository.
2. Resolve the GitHub repository and default branch:

```bash
gh repo view --json nameWithOwner,url,defaultBranchRef
```

3. If the repository or GitHub host is unclear, ask the user.
4. If `gh` is not authenticated, report the error and tell the user what is required. Do not run `gh auth login`.

Completion criterion: the GitHub repository, host, and default branch are known.

## Check the branch

1. Read the current branch and working-tree status.
2. If the current branch is the default branch, stop and tell the user to create or switch to a feature branch first.
3. If uncommitted changes exist, stop and tell the user to commit them first. Use the `commit-and-push` skill when appropriate.
4. Check whether the current branch already has a pull request with `gh pr view --json number,url,state,title`.
5. If a pull request already exists, show its URL and stop. Do not create a duplicate.

Completion criterion: the current branch is a clean, non-default branch with no existing pull request.

## Push the branch

If the branch is not pushed or has no upstream, push it with:

```bash
git push --set-upstream origin <branch>
```

Use the existing upstream when one is already configured. Do not force-push or rewrite history.

Completion criterion: the current branch is available on the GitHub remote.

## Build the pull request

Create a clear title and a body with these sections:

```markdown
## Summary

## Changes

## Checks

## Notes
```

Omit empty sections. Include check results only when the repository defines checks or the user supplies results. Link an issue only when the user provides one.

Create a regular open pull request with the detected default branch as the base:

```bash
gh pr create \
  --base <default-branch> \
  --head <current-branch> \
  --title '<title>' \
  --body '<body>'
```

Do not use `--draft`. Do not add reviewers or labels unless the user requests them.

Completion criterion: one regular open pull request exists for the current branch, and its URL is reported to the user.
