---
name: pr-description
description: Generate a PR description for the current branch. Use when asked to write a PR, open a pull request, or describe changes.
---

# ionstudioapps PR Description

Generate a PR description by analysing the current branch's commits and diff against `main`.

## Steps

1. Run `git log main..HEAD --oneline` to see all commits on this branch
2. Run `git diff main..HEAD --stat` to see changed files
3. Read the key changed files to understand what actually changed
4. Write the PR description using the template below
5. Create the PR with `gh pr create --title "..." --body "..."`

## Template

```
## What

<1-3 bullet points describing what changed — focus on the feature/fix, not the implementation>

## Why

<1-2 sentences on the motivation — bug fix, feature request, refactor, etc.>

## How to test

- [ ] <specific step to verify the main change>
- [ ] <edge case or secondary behaviour to check>
- [ ] <regression: verify existing behaviour still works>

## Screenshots (if UI changes)

<add before/after screenshots for any visual changes>

## Notes

<anything reviewers should know: breaking changes, follow-up tickets, known issues>
```

## PR title format

- New feature: `add <feature name>`
- Bug fix: `fix <what was broken>`
- Refactor: `refactor <what changed>`
- Chore: `chore <what was done>`

Keep the title under 60 characters. No ticket numbers in the title.

## Repo context
- `ionstudioapps/wheeltodo` — Next.js web + Expo mobile monorepo
- `ionstudioapps/rocky` — Slack/Notion/Google Calendar bot (Node.js)
