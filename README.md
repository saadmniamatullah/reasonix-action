# Reasonix Agent

A reusable GitHub Action that uses [Reasonix](https://reasonix.io) to automatically fix issues and open PRs.

Comment `@reasonix` on any issue to trigger the agent. It reads the issue, implements the fix, and opens a PR. Merging the PR closes the issue.

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

jobs:
  reasonix:
    if: >-
      github.event.issue.pull_request == null &&
      startsWith(github.event.comment.body, '@reasonix')
    uses: saadmniamatullah/reasonix-action/.github/workflows/reasonix.yml@main
    secrets:
      DEEPSEEK_API_KEY: ${{ secrets.DEEPSEEK_API_KEY }}
    permissions:
      contents: write
      issues: write
      pull-requests: write
```

### 3. Comment on an issue

```
@reasonix fix the login bug described in this issue
```

## What happens

1. Reasonix clones the repo and reads the issue
2. It implements the fix with focused commits
3. A PR is opened against the default branch
4. Merging the PR closes the issue (via `Fixes #N`)

## Customization

### Cost cap

The default budget is $2.00 per run. To change it, fork this repo and edit the `--budget` flag in `.github/workflows/reasonix.yml`.

### Prompt

To customize the prompt, fork this repo and modify the `PROMPT` env var.
