# Quick Start Guide - 1215-Labs Centralized Workflows

## 🚀 Get Started in 3 Steps

### 1. Choose Your Workflow Type

Pick from our starter templates or call reusable workflows directly.

### 2. Add to Your Repository

Create `.github/workflows/ci.yml` in your repository:

#### For Node.js Projects:
```yaml
name: CI

on: [push, pull_request]

jobs:
  ci:
    uses: 1215-Labs/.github/.github/workflows/reusable-ci.yml@main
    with:
      language: 'nodejs'
      node-version: '18'
      run-tests: true
      run-lint: true
```

#### For Python Projects:
```yaml
name: CI

on: [push, pull_request]

jobs:
  ci:
    uses: 1215-Labs/.github/.github/workflows/reusable-ci.yml@main
    with:
      language: 'python'
      python-version: '3.11'
      run-tests: true
      run-lint: true
```

#### For Docker + Deployment:
```yaml
name: Deploy

on:
  push:
    branches: [ main ]

jobs:
  build:
    uses: 1215-Labs/.github/.github/workflows/reusable-docker-build.yml@main
    with:
      image-name: ${{ github.repository }}
      dockerfile-path: './Dockerfile'
    secrets:
      registry-username: ${{ github.actor }}
      registry-password: ${{ secrets.GITHUB_TOKEN }}

  deploy:
    needs: build
    uses: 1215-Labs/.github/.github/workflows/reusable-deploy.yml@main
    with:
      environment: 'staging'
      deployment-command: 'kubectl apply -f k8s/'
      smoke-test-url: 'https://staging.example.com/health'
    secrets:
      deployment-token: ${{ secrets.STAGING_DEPLOY_TOKEN }}
```

### 3. Configure Secrets (if needed)

**For Deployments:**
- Go to repository Settings → Secrets and variables → Actions
- Add: `STAGING_DEPLOY_TOKEN`, `PRODUCTION_DEPLOY_TOKEN`
- Optional: `SLACK_WEBHOOK_URL` for notifications

**For GitHub Container Registry (GHCR):**
- No setup needed! `GITHUB_TOKEN` is automatically available

## 📚 Learn More

- **[WORKFLOWS.md](WORKFLOWS.md)** - Complete documentation
- **[examples/](examples/)** - Real-world examples
- **[CONTRIBUTING.md](CONTRIBUTING.md)** - How to contribute

## 🔗 Available Workflows

| Workflow | Purpose | Call With |
|----------|---------|-----------|
| CI | Testing & linting | `reusable-ci.yml` |
| Security Scan | CodeQL, dependencies, secrets | `reusable-security-scan.yml` |
| Docker Build | Multi-platform container builds | `reusable-docker-build.yml` |
| Deploy | Environment-aware deployment | `reusable-deploy.yml` |

## 🧩 Available Actions

| Action | Purpose | Use With |
|--------|---------|----------|
| setup-nodejs-environment | Setup Node.js + deps | `1215-Labs/.github/.github/actions/setup-nodejs-environment@main` |
| setup-python-environment | Setup Python + deps | `1215-Labs/.github/.github/actions/setup-python-environment@main` |
| notify-slack | Send Slack notifications | `1215-Labs/.github/.github/actions/notify-slack@main` |

## 💡 Pro Tips

1. **Use workflow templates** - They appear in your Actions tab automatically
2. **Pin to @main** for latest features or use tags for stability
3. **Set up GitHub Environments** for production deployments with protection rules
4. **Configure organization-level secrets** to share across repositories
5. **Check the examples/** directory for complete working examples

## ❓ Need Help?

- 📖 Read [WORKFLOWS.md](WORKFLOWS.md) for detailed documentation
- 🐛 [Open an issue](https://github.com/1215-Labs/.github/issues)
- 💬 [Start a discussion](https://github.com/1215-Labs/.github/discussions)

---

**Ready to automate?** Start with a simple CI workflow and expand from there! 🎉
