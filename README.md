# 1215-Labs GitHub AI Workflows (`.github`)

This repository contains **comment-driven GitHub Actions workflows** that run AI agents (Claude / Codex / Cursor) to:

- Create automated **issue fixes** (write access)
- Perform automated **PR reviews** (read-only + comment)
- Generate **release notes**

These workflows are designed to be **reusable across repositories**. See `WORKFLOWS_SETUP.md` for the full setup checklist and org-level scaling guidance.

## What’s included

- **Workflows**: `.github/workflows/*.yml`
  - Fix: `claude-fix.yml`, `codex-fix.yml`, `cursor-fix.yml`
  - Review: `claude-review.yml`, `codex-review.yml`, `cursor-review.yml`
  - Release notes: `release-notes.yml`
- **Prompt templates**:
  - `.github/issue_fix_prompt.md`
  - `.github/pr_review_prompt.md`
  - `.github/pull_request_template.md`

## How to trigger

Create an issue or PR, then post a **new comment** containing one of the trigger tags:

- `@claude-fix` / `@claude-review`
- `@codex-fix` / `@codex-review`
- `@cursor-fix` / `@cursor-review`

The workflow will only run if the commenter is allowlisted (see below).

## Required configuration

### 1) Allowlist (required)

Create an Actions variable named:

- `AI_WORKFLOW_AUTHORIZED_USERS_JSON`
  - Value: JSON array of GitHub usernames, e.g. `["your-handle","teammate1"]`

This can be set at the **repo level** or **organization level**.

### 2) Secrets (required for the workflows you enable)

Add the relevant Actions secret(s):

- **Claude workflows**: `CLAUDE_CODE_OAUTH_TOKEN`
- **Codex workflows**: `OPENAI_API_KEY`
- **Cursor workflows**: `CURSOR_API_KEY`

Secrets can be stored at the **repo level** or **organization level**.

## Repo vs org-level setup

For the detailed steps (including org-level scaling and reusable workflow caveats), read:

- `WORKFLOWS_SETUP.md`

GitHub reference:

- [Using secrets in GitHub Actions](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets)

## Security notes

- These workflows restrict execution using `AI_WORKFLOW_AUTHORIZED_USERS_JSON`.
- `GITHUB_TOKEN` is used for GitHub API access inside workflows (exposed as `GH_TOKEN` for `gh`).
- For cases where actions should run as a real user instead of `github-actions[bot]`, consider using a fine-grained PAT and updating the workflow(s) accordingly (see `WORKFLOWS_SETUP.md`).


