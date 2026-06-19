# Reasonix Agent

A reusable GitHub Action that uses [Reasonix](https://reasonix.io) to automatically fix issues and open PRs.

Comment `@reasonix` on any issue or PR to trigger the agent. You can also add the `reasonix` label to an issue to trigger it without commenting.

- **On issues:** Reads the issue, implements the fix, opens a PR
- **On PRs:** Reads the comment, pushes commits to the existing PR branch

## Setup

### 1. Add the secret

Add `DEEPSEEK_API_KEY` to each consumer repo (Settings → Secrets and variables → Actions).

### 2. Add the workflow

Create `.github/workflows/reasonix.yml` in the consumer repo:

```yaml
name: Reasonix Agent
on:
  issue_comment:
    types: [created]
  issues:
    types: [opened]

jobs:
  reasonix:
    if: >-
      (github.event_name == 'issue_comment' && startsWith(github.event.comment.body, '@reasonix')) ||
      (github.event_name == 'issues' && contains(github.event.issue.labels.*.name, 'reasonix'))
    uses: saadmniamatullah/reasonix-action/.github/workflows/reasonix.yml@main
    with:
      issue-number: ${{ github.event.issue.number }}
      issue-title: ${{ github.event.issue.title }}
      issue-body: ${{ github.event.issue.body }}
      comment-id: ${{ github.event_name == 'issue_comment' && github.event.comment.id || 0 }}
      comment-body: ${{ github.event_name == 'issue_comment' && github.event.comment.body || '' }}
      is-pr: ${{ github.event.issue.pull_request != null }}
      pr-branch: ${{ github.event.issue.pull_request != null && github.event.issue.pull_request.head.ref || '' }}
    secrets:
      DEEPSEEK_API_KEY: ${{ secrets.DEEPSEEK_API_KEY }}
    permissions:
      contents: write
      issues: write
      pull-requests: write
```

### 3. Trigger reasonix

**Comment on an issue or PR:**

```
@reasonix fix the login bug described in this issue
```

**Or add the `reasonix` label to an issue** — no comment needed.

## What happens

### On issues

1. 👀 Eyes reaction added to your comment (if triggered by comment)
2. Reasonix clones the repo and reads the issue
3. It implements the fix with focused commits
4. A PR is opened against the default branch
5. Merging the PR closes the issue (via `Fixes #N`)

### On PRs

1. 👀 Eyes reaction added to your comment
2. Reasonix checks out the PR branch
3. It reads the PR description and your comment
4. New commits are pushed to the existing PR branch
5. The PR updates automatically with the new changes

## Customization

### Cost cap

The default budget is $2.00 per run. To change it, fork this repo and edit the `--budget` flag in `.github/workflows/reasonix.yml`.

### Prompt

To customize the prompt, fork this repo and modify the `PROMPT` env var.

### Project conventions (AGENTS.md)

Reasonix reads `AGENTS.md` from your repo root at startup. Use this to enforce project-specific conventions:

```markdown
<!-- AGENTS.md -->

## Code style

- Format all code with biome before committing: `npx biome check --write`
- Run tests before committing: `npm test`
- Use conventional commits (fix:, feat:, chore:, etc.)
```

This keeps the workflow generic while letting each project enforce its own standards.
