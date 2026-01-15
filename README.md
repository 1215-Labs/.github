# 1215-Labs Organization `.github` Repository

This repository serves as the central hub for **reusable workflows, composite actions, and automation** across all 1215-Labs repositories.

## What's Available

### 🤖 AI-Powered Workflows
Comment-driven GitHub Actions workflows that run AI agents (Claude / Codex / Cursor) to:

- Create automated **issue fixes** (write access)
- Perform automated **PR reviews** (read-only + comment)
- Generate **release notes**

### 🔄 Reusable CI/CD Workflows
Production-ready workflows for common development tasks:

- **CI Pipeline** - Node.js and Python testing with linting and coverage
- **Security Scanning** - CodeQL, dependency checks, and secret detection
- **Docker Build** - Multi-platform container builds with vulnerability scanning
- **Deployment** - Environment-aware deployments with smoke tests

### 🧩 Composite Actions
Pre-built action steps for common setup tasks:

- Setup Node.js environment with automatic package manager detection
- Setup Python environment with pip/poetry/pipenv support
- Slack notifications for workflow status

### 📝 Workflow Templates
Starter templates that appear in your repository's Actions tab for quick setup.

## Quick Start

1. **Choose a workflow approach:**
   - Use [workflow templates](#features) from your repository's Actions tab
   - Or call [reusable workflows](#features) directly in your workflow files

2. **For AI workflows**, trigger by posting a comment:
   - `@claude-fix` / `@claude-review`
   - `@codex-fix` / `@codex-review`
   - `@cursor-fix` / `@cursor-review`

3. **Configure required secrets** (see [Configuration](#configuration) below)

### Using Reusable Workflows

Add to your repository's `.github/workflows/ci.yml`:

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    uses: 1215-Labs/.github/.github/workflows/reusable-ci.yml@main
    with:
      language: 'nodejs'
      node-version: '18'
      run-tests: true
      run-lint: true
```

## Documentation

🚀 **[QUICKSTART.md](QUICKSTART.md)** - Get started in 3 steps (perfect for beginners)

📖 **[WORKFLOWS.md](WORKFLOWS.md)** - Complete documentation for the centralized workflow system:
- Detailed guide for all reusable workflows
- Composite action references
- Configuration examples
- Best practices and troubleshooting

📋 **[WORKFLOWS_SETUP.md](WORKFLOWS_SETUP.md)** - AI workflow setup guide:
- Authorization and allowlisting
- Organization-level secrets configuration
- Reusable workflow patterns

🤝 **[CONTRIBUTING.md](CONTRIBUTING.md)** - Contribution guidelines

📜 **[CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)** - Community standards

## Configuration

### AI Workflows

#### Allowlist (Required)
Set the Actions variable `AI_WORKFLOW_AUTHORIZED_USERS_JSON`:
```json
["your-handle","teammate1"]
```

#### Secrets (Required)
- **Claude**: `CLAUDE_CODE_OAUTH_TOKEN`
- **Codex**: `OPENAI_API_KEY`
- **Cursor**: `CURSOR_API_KEY`

### CI/CD Workflows

#### For Docker Builds
- `GITHUB_TOKEN` (automatically provided)

#### For Deployments
- `STAGING_DEPLOY_TOKEN`
- `PRODUCTION_DEPLOY_TOKEN`
- `SLACK_WEBHOOK_URL` (optional)

**Tip**: Configure secrets at the organization level for easier management across repositories.

## Repository Structure

```
.github/
├── workflows/              # Reusable and AI-driven workflows
│   ├── reusable-ci.yml
│   ├── reusable-security-scan.yml
│   ├── reusable-docker-build.yml
│   ├── reusable-deploy.yml
│   ├── claude-fix.yml
│   ├── claude-review.yml
│   └── ...
├── actions/                # Composite actions
│   ├── setup-nodejs-environment/
│   ├── setup-python-environment/
│   └── notify-slack/
├── issue_fix_prompt.md     # AI fix prompt template
├── pr_review_prompt.md     # AI review prompt template
└── pull_request_template.md
workflow-templates/         # Starter templates
├── ci-nodejs.yml
├── ci-python.yml
├── security-scan.yml
└── docker-deploy.yml
examples/                   # Usage examples
├── example-nodejs-ci.yml
├── example-python-ci.yml
└── example-full-pipeline.yml
```

## Features

### Reusable Workflows

✅ **CI Workflow** - Automated testing and linting for Node.js and Python  
✅ **Security Scan** - CodeQL, dependency scanning, secret detection  
✅ **Docker Build** - Multi-platform container builds with caching  
✅ **Deployment** - Environment-aware deployments with smoke tests

### Composite Actions

✅ **Setup Node.js** - Auto-detects npm, yarn, or pnpm  
✅ **Setup Python** - Supports pip, poetry, and pipenv  
✅ **Slack Notify** - Rich workflow notifications

### Best Practices Built-in

✅ Dependency caching for faster builds  
✅ Concurrency control to cancel outdated runs  
✅ Security scanning and vulnerability detection  
✅ Artifact management and retention  
✅ Multi-platform support  
✅ Comprehensive error handling

## Examples

See the `examples/` directory for complete, working examples:

- **example-nodejs-ci.yml** - Node.js continuous integration
- **example-python-ci.yml** - Python continuous integration
- **example-full-pipeline.yml** - Complete CI/CD pipeline with Docker and deployment

## Security

- AI workflows restrict execution using allowlists
- All workflows follow principle of least privilege
- Secrets are never logged or exposed
- GitHub Environments provide approval gates for production deploys
- Dependabot keeps action versions up to date

For security issues, please contact the 1215-Labs security team.

## Support

- 📖 Read [WORKFLOWS.md](WORKFLOWS.md) for detailed documentation
- 🐛 [Open an issue](https://github.com/1215-Labs/.github/issues) for bugs
- 💡 [Start a discussion](https://github.com/1215-Labs/.github/discussions) for questions

## License

Available for use by all 1215-Labs organization repositories.

---

**Maintained by**: 1215-Labs DevOps Team
