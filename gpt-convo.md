
## Tag-to-run behavior (what you want)

Yes—your goal is exactly how these workflows are designed to work:

- Comment `@claude-fix` → runs `claude-fix.yml`
- Comment `@claude-review` → runs `claude-review.yml`
- Comment `@codex-fix` → runs `codex-fix.yml`
- Comment `@codex-review` → runs `codex-review.yml`
- Comment `@cursor-fix` → runs `cursor-fix.yml`
- Comment `@cursor-review` → runs `cursor-review.yml`

### Where the comment must be posted

The workflows trigger on **new comments**:

- Issue comments (`on: issue_comment`)
- PR review comments (`on: pull_request_review_comment`)

So: put the tag in a brand-new comment body (like your screenshot), not just in the issue/PR description.

### What must be configured (or the run will be skipped/blocked)

- `AI_WORKFLOW_AUTHORIZED_USERS_JSON` must include your GitHub username (as an Actions variable, repo-level or org-level).
- The relevant API secret must be set:
  - Claude: `CLAUDE_CODE_OAUTH_TOKEN`
  - Codex: `OPENAI_API_KEY`
  - Cursor: `CURSOR_API_KEY`

### Common “why didn’t it run?” gotchas

- **Not allowlisted**: your username isn’t in `AI_WORKFLOW_AUTHORIZED_USERS_JSON` (workflow will post an “unauthorized” message).
- **Secret not available**: e.g. org secret not granted to the repo, or secrets not provided for certain event sources (see GitHub docs: [Using secrets in GitHub Actions](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets)).

