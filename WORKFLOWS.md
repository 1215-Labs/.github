# 1215-Labs Centralized Workflow System

This documentation describes the comprehensive, reusable workflow system available to all repositories in the 1215-Labs organization.

## Table of Contents

- [Overview](#overview)
- [Reusable Workflows](#reusable-workflows)
- [Composite Actions](#composite-actions)
- [Workflow Templates](#workflow-templates)
- [Getting Started](#getting-started)
- [Required Secrets and Variables](#required-secrets-and-variables)
- [Best Practices](#best-practices)
- [Examples](#examples)
- [Troubleshooting](#troubleshooting)

## Overview

The `.github` repository contains centralized CI/CD workflows, composite actions, and templates that can be reused across all 1215-Labs repositories. This system provides:

- **Standardized CI/CD pipelines** for Node.js and Python projects
- **Security scanning** with CodeQL, dependency checks, and secret detection
- **Docker build and deployment** workflows
- **Composite actions** for common setup tasks
- **Starter templates** that appear in the GitHub Actions tab

## Reusable Workflows

Reusable workflows are complete workflow jobs that can be called from other repositories. They use `workflow_call` and can accept inputs and secrets.

### 1. Reusable CI Workflow

**Location**: `.github/workflows/reusable-ci.yml`

General-purpose CI workflow that supports both Node.js and Python projects.

#### Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `language` | string | Yes | - | Primary language (`nodejs` or `python`) |
| `node-version` | string | No | `18` | Node.js version to use |
| `python-version` | string | No | `3.11` | Python version to use |
| `run-tests` | boolean | No | `true` | Whether to run tests |
| `run-lint` | boolean | No | `true` | Whether to run linting |
| `working-directory` | string | No | `.` | Working directory for commands |

#### Example Usage

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  ci:
    uses: 1215-Labs/.github/.github/workflows/reusable-ci.yml@main
    with:
      language: 'nodejs'
      node-version: '18'
      run-tests: true
      run-lint: true
```

#### Features

- Automatic package manager detection (npm, yarn, pnpm, pip, poetry, pipenv)
- Dependency caching for faster builds
- Test execution with coverage reporting
- Linting with popular tools (ESLint, Prettier, flake8, pylint, black)
- Coverage upload to Codecov
- Artifact retention for 30 days

---

### 2. Reusable Security Scan Workflow

**Location**: `.github/workflows/reusable-security-scan.yml`

Comprehensive security scanning workflow.

#### Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `language` | string | No | `javascript` | Language for CodeQL analysis |
| `severity-threshold` | string | No | `high` | Minimum severity to fail build |

#### Example Usage

```yaml
name: Security Scan

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 9 * * 1'  # Weekly on Mondays

jobs:
  security:
    uses: 1215-Labs/.github/.github/workflows/reusable-security-scan.yml@main
    with:
      language: 'javascript'
      severity-threshold: 'high'
```

#### Features

- **CodeQL Analysis**: Static code analysis for security vulnerabilities
- **Dependency Scanning**: npm audit and pip-audit for known vulnerabilities
- **Secret Scanning**: Gitleaks and TruffleHog for exposed secrets
- **Security Reports**: Automated summary and artifact uploads

---

### 3. Reusable Docker Build Workflow

**Location**: `.github/workflows/reusable-docker-build.yml`

Build, tag, and push Docker images to container registries.

#### Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `image-name` | string | Yes | - | Name of the Docker image |
| `dockerfile-path` | string | No | `./Dockerfile` | Path to Dockerfile |
| `registry` | string | No | `ghcr.io` | Container registry URL |
| `context` | string | No | `.` | Build context path |
| `platforms` | string | No | `linux/amd64` | Target platforms |

#### Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `registry-username` | Yes | Container registry username |
| `registry-password` | Yes | Container registry password/token |

#### Example Usage

```yaml
name: Docker Build

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]

jobs:
  build:
    uses: 1215-Labs/.github/.github/workflows/reusable-docker-build.yml@main
    with:
      image-name: ${{ github.repository }}
      dockerfile-path: './Dockerfile'
      registry: 'ghcr.io'
      platforms: 'linux/amd64,linux/arm64'
    secrets:
      registry-username: ${{ github.actor }}
      registry-password: ${{ secrets.GITHUB_TOKEN }}
```

#### Features

- Multi-platform builds (amd64, arm64)
- Automatic tagging (branch, SHA, semver)
- Docker layer caching for faster builds
- SBOM and provenance attestation
- Vulnerability scanning with Trivy
- Push to any container registry (GHCR, Docker Hub, ECR, etc.)

---

### 4. Reusable Deploy Workflow

**Location**: `.github/workflows/reusable-deploy.yml`

Generic deployment workflow with environment protection and smoke tests.

#### Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `environment` | string | Yes | - | Environment (`staging` or `production`) |
| `deployment-command` | string | Yes | - | Command to execute for deployment |
| `working-directory` | string | No | `.` | Working directory |
| `smoke-test-url` | string | No | - | URL for smoke tests |
| `smoke-test-enabled` | boolean | No | `true` | Whether to run smoke tests |
| `slack-webhook-enabled` | boolean | No | `false` | Enable Slack notifications |

#### Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `deployment-token` | Yes | Authentication token for deployment |
| `slack-webhook-url` | No | Slack webhook (if notifications enabled) |

#### Example Usage

```yaml
name: Deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    uses: 1215-Labs/.github/.github/workflows/reusable-deploy.yml@main
    with:
      environment: 'staging'
      deployment-command: 'kubectl apply -f k8s/staging/'
      smoke-test-url: 'https://staging.example.com/health'
      smoke-test-enabled: true
    secrets:
      deployment-token: ${{ secrets.STAGING_DEPLOY_TOKEN }}
```

#### Features

- GitHub Environment protection rules
- Concurrency control (prevents overlapping deployments)
- Automatic smoke tests with retry logic
- Slack notifications on success/failure
- Deployment summary in GitHub UI

---

## Composite Actions

Composite actions are reusable steps that can be included in any workflow.

### 1. Setup Node.js Environment

**Location**: `.github/actions/setup-nodejs-environment/action.yml`

Checks out code, sets up Node.js, and installs dependencies.

#### Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `node-version` | string | No | `18` | Node.js version |
| `working-directory` | string | No | `.` | Working directory |
| `registry-url` | string | No | - | npm registry URL |
| `cache` | string | No | `true` | Enable caching |

#### Example Usage

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: 1215-Labs/.github/.github/actions/setup-nodejs-environment@main
        with:
          node-version: '18'
          working-directory: './frontend'
```

---

### 2. Setup Python Environment

**Location**: `.github/actions/setup-python-environment/action.yml`

Checks out code, sets up Python, and installs dependencies.

#### Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `python-version` | string | No | `3.11` | Python version |
| `working-directory` | string | No | `.` | Working directory |
| `cache` | string | No | `true` | Enable caching |
| `install-dev-dependencies` | string | No | `true` | Install dev dependencies |

#### Example Usage

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: 1215-Labs/.github/.github/actions/setup-python-environment@main
        with:
          python-version: '3.11'
          working-directory: './backend'
```

---

### 3. Notify Slack

**Location**: `.github/actions/notify-slack/action.yml`

Sends workflow status notifications to Slack.

#### Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `webhook-url` | string | Yes | - | Slack webhook URL |
| `status` | string | Yes | - | Workflow status |
| `message` | string | No | - | Custom message |
| `job-name` | string | No | `GitHub Actions` | Job name |
| `include-commit-info` | string | No | `true` | Include commit info |

#### Example Usage

```yaml
jobs:
  notify:
    runs-on: ubuntu-latest
    if: always()
    steps:
      - uses: 1215-Labs/.github/.github/actions/notify-slack@main
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK_URL }}
          status: ${{ job.status }}
          job-name: 'CI Pipeline'
```

---

## Workflow Templates

Workflow templates appear in the "Actions" tab when creating a new workflow in GitHub repositories.

### Available Templates

1. **ci-nodejs.yml** - Node.js CI starter
2. **ci-python.yml** - Python CI starter
3. **security-scan.yml** - Security scanning starter
4. **docker-deploy.yml** - Docker build and deploy pipeline

These templates automatically appear in your repository's Actions tab under "By 1215-Labs".

---

## Getting Started

### Quick Start for New Repository

1. **Choose a template** from the Actions tab in your repository
2. **Customize the inputs** (versions, paths, etc.)
3. **Set up required secrets** (see below)
4. **Commit and push** the workflow file
5. **Watch your workflow run** on the Actions tab

### Manual Setup

1. Create `.github/workflows/` directory in your repository
2. Create a workflow file (e.g., `ci.yml`)
3. Call the reusable workflow:

```yaml
name: CI

on: [push, pull_request]

jobs:
  ci:
    uses: 1215-Labs/.github/.github/workflows/reusable-ci.yml@main
    with:
      language: 'nodejs'
      node-version: '18'
```

---

## Required Secrets and Variables

### Repository Secrets

Configure these in: **Settings → Secrets and variables → Actions → Secrets**

#### For Docker Workflows
- `GITHUB_TOKEN` (automatically provided)
- Custom registry credentials if not using GHCR

#### For Deployment Workflows
- `STAGING_DEPLOY_TOKEN` - Deployment token for staging
- `PRODUCTION_DEPLOY_TOKEN` - Deployment token for production
- `SLACK_WEBHOOK_URL` - Slack webhook for notifications (optional)

### Organization-Level Secrets (Recommended)

For better management across multiple repositories, configure secrets at the organization level:

**Settings → Security → Secrets and variables → Actions**

Then scope them to specific repositories or all repositories.

### GitHub Environments

For the deploy workflow, set up environments in your repository:

**Settings → Environments → New environment**

Create `staging` and `production` environments with:
- Protection rules (required reviewers for production)
- Environment secrets
- Deployment branches rules

---

## Best Practices

### 1. Version Pinning

Always pin reusable workflows to a specific version or commit:

```yaml
# ✅ Good - pinned to main branch
uses: 1215-Labs/.github/.github/workflows/reusable-ci.yml@main

# ✅ Better - pinned to specific tag
uses: 1215-Labs/.github/.github/workflows/reusable-ci.yml@v1.0.0

# ✅ Best - pinned to commit SHA
uses: 1215-Labs/.github/.github/workflows/reusable-ci.yml@abc123
```

### 2. Concurrency Control

Use concurrency groups to cancel outdated runs:

```yaml
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
```

### 3. Timeout Settings

Always set timeouts to prevent stuck workflows:

```yaml
jobs:
  ci:
    timeout-minutes: 30
```

### 4. Matrix Testing

Test multiple versions in parallel:

```yaml
jobs:
  test:
    strategy:
      matrix:
        node-version: [16, 18, 20]
    uses: 1215-Labs/.github/.github/workflows/reusable-ci.yml@main
    with:
      language: 'nodejs'
      node-version: ${{ matrix.node-version }}
```

### 5. Security

- Never hardcode secrets in workflows
- Use GitHub Environments for sensitive deployments
- Enable branch protection rules
- Require status checks before merging
- Use `GITHUB_TOKEN` permissions sparingly

### 6. Performance

- Use caching for dependencies
- Cancel outdated workflow runs
- Use matrix strategies for parallel execution
- Minimize artifact sizes
- Set appropriate retention periods

---

## Examples

See the `examples/` directory for complete example workflows:

- [`examples/example-nodejs-ci.yml`](examples/example-nodejs-ci.yml) - Node.js CI
- [`examples/example-python-ci.yml`](examples/example-python-ci.yml) - Python CI
- [`examples/example-full-pipeline.yml`](examples/example-full-pipeline.yml) - Complete CI/CD pipeline

---

## Troubleshooting

### Common Issues

#### 1. Workflow not found

**Error**: `Unable to resolve action 1215-Labs/.github/.github/workflows/...`

**Solution**:
- Verify the path is correct (note the double `.github`)
- Check that the workflow exists in the `.github` repository
- Ensure you're using `@main` or a valid ref

#### 2. Permission denied

**Error**: `Resource not accessible by integration`

**Solution**:
- Check workflow permissions in the calling workflow
- Add required permissions:
  ```yaml
  permissions:
    contents: read
    pull-requests: write
  ```

#### 3. Secrets not available

**Error**: `Secret not found`

**Solution**:
- Verify secret is set in repository or organization settings
- Ensure secret is scoped to the repository
- Pass secrets explicitly in the calling workflow:
  ```yaml
  secrets:
    deployment-token: ${{ secrets.DEPLOYMENT_TOKEN }}
  ```

#### 4. Workflow not appearing in Actions tab

**Solution**:
- Ensure `.properties.json` file exists alongside template
- Check JSON syntax is valid
- Verify file is in `workflow-templates/` directory
- Wait a few minutes for GitHub to refresh

#### 5. Cache not working

**Solution**:
- Verify `cache` input is set to `true`
- Check that lock files exist (`package-lock.json`, `requirements.txt`)
- Review cache key in action logs
- Note: Cache is restored from base branch first

### Getting Help

1. **Check the workflow logs** in the Actions tab
2. **Review this documentation** for correct usage
3. **Check example workflows** in the `examples/` directory
4. **Create an issue** in the `.github` repository
5. **Contact the DevOps team** for assistance

---

## Maintenance and Updates

### Keeping Workflows Up to Date

This repository uses Dependabot to automatically update action versions. Review and merge Dependabot PRs regularly.

### Making Changes

When updating reusable workflows:

1. Create a feature branch
2. Make and test your changes
3. Update this documentation
4. Create a PR for review
5. Tag a new version after merging

### Versioning Strategy

- `@main` - Latest stable version
- `@v1`, `@v2` - Major versions
- `@v1.2.3` - Specific versions
- `@commit-sha` - Specific commits

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on contributing to this workflow system.

## License

These workflows are available for use by all 1215-Labs repositories.

---

**Last Updated**: 2026-01-15
**Maintained by**: 1215-Labs DevOps Team
