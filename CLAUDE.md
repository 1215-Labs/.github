# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **template repository** containing reusable GitHub Actions workflows for AI-powered automation. It enables comment-driven AI agents (Claude, Codex, Cursor) to:
- Fix issues automatically (write access)
- Review pull requests (read-only + commenting)
- Generate release notes

This repo is designed to be copied into other repositories. There are no build/test/lint commands—this is purely configuration and YAML workflow files.

## Repository Structure

```
.github/
├── workflows/           # GitHub Actions workflow files
│   ├── claude-fix.yml   # @claude-fix trigger (write)
│   ├── claude-review.yml # @claude-review trigger (read-only)
│   ├── codex-fix.yml    # @codex-fix trigger (write)
│   ├── codex-review.yml # @codex-review trigger (read-only)
│   ├── cursor-fix.yml   # @cursor-fix trigger (write)
│   ├── cursor-review.yml # @cursor-review trigger (read-only)
│   └── release-notes.yml # Manual trigger for release notes
├── issue_fix_prompt.md      # Template for fix operations
├── pr_review_prompt.md      # Template for review operations
└── pull_request_template.md # PR template
```

## Architecture

### Trigger Flow
1. User posts comment with `@{AI}-{action}` tag (e.g., `@claude-fix`)
2. Workflow checks if commenter is in `AI_WORKFLOW_AUTHORIZED_USERS_JSON` allowlist
3. If authorized: Loads prompt template, prepends GitHub context, runs AI agent
4. If unauthorized: Posts rejection message

### Authorization System
- **Variable**: `AI_WORKFLOW_AUTHORIZED_USERS_JSON`
- **Format**: JSON array of GitHub usernames, e.g., `["username1","username2"]`
- **Location**: Repository or organization-level Actions variable
- **Default**: Empty array `[]` (no users authorized) — **you must configure this variable**
- **Security**: Workflows fail closed; without explicit configuration, all triggers are denied

### Prompt Template Strategy
- Templates use `{AI_ASSISTANT}` placeholder, replaced via `sed` at runtime
- GitHub context (repo, issue/PR number, triggering user) is prepended
- Templates are searched in two locations: `.github/` (primary) or repo root (for copied repos)

### Permissions Model
**Fix workflows (write):**
- `contents: write` - create branches, edit files
- `pull-requests: write` - create/update PRs
- `issues: write` - comment on issues

**Review workflows (read-only):**
- `contents: read` - read repo files
- `pull-requests: write` - post comments only
- `issues: write` - post comments only

### Required Secrets
- **Claude**: `CLAUDE_CODE_OAUTH_TOKEN`
- **Codex**: `OPENAI_API_KEY`
- **Cursor**: `CURSOR_API_KEY`

### Optional Variables (Release Notes)
For `release-notes.yml`, configure these repository variables:
- `RELEASE_NOTES_PRODUCT_NAME` — Product name (default: repository name)
- `RELEASE_NOTES_PRODUCT_DESCRIPTION` — Product description (default: "a software project")
- `RELEASE_NOTES_FRONTEND_DIR` — Frontend directory path for diff stats (optional)
- `RELEASE_NOTES_BACKEND_DIR` — Backend directory path for diff stats (optional)

## Key Patterns

### Workflow Conditionals
Authorization is checked inline using:
```yaml
contains(fromJSON((startsWith(vars.AI_WORKFLOW_AUTHORIZED_USERS_JSON, '[') && vars.AI_WORKFLOW_AUTHORIZED_USERS_JSON) || '[]'), github.event.comment.user.login)
```

### Loading Instructions
Workflows use heredocs to preserve multiline prompt content:
```yaml
echo "CUSTOM_INSTRUCTIONS<<EOF" >> $GITHUB_OUTPUT
```

### Bot Attribution
- Claude workflows: Uses `github-actions[bot]` via GITHUB_TOKEN
- Codex/Cursor: Set `git config user.name "{ai}-bot"`

## Reusability

When copying to another repo:
1. Copy `.github/workflows/*.yml` and prompt templates
2. **Required**: Set `AI_WORKFLOW_AUTHORIZED_USERS_JSON` variable (workflows deny all by default)
3. Add required secrets for enabled workflows
4. Optionally configure release notes variables (`RELEASE_NOTES_*`)
5. No YAML modifications needed if names match

For org-level scaling, see `WORKFLOWS_SETUP.md` for reusable workflow patterns with `workflow_call`.
