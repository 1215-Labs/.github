# Contributing to 1215-Labs Workflows

Thank you for your interest in contributing to the 1215-Labs centralized workflow system! This document provides guidelines and instructions for contributing.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How Can I Contribute?](#how-can-i-contribute)
- [Development Setup](#development-setup)
- [Contribution Guidelines](#contribution-guidelines)
- [Style Guidelines](#style-guidelines)
- [Testing Your Changes](#testing-your-changes)
- [Submitting Changes](#submitting-changes)

## Code of Conduct

This project adheres to our [Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code.

## How Can I Contribute?

### Reporting Bugs

Before creating bug reports, please check existing issues to avoid duplicates. When creating a bug report, include:

- **Clear title and description**
- **Steps to reproduce** the issue
- **Expected vs actual behavior**
- **Workflow logs** (sanitize any sensitive information)
- **Repository where the issue occurred** (if applicable)
- **Version or commit SHA** of the workflow being used

### Suggesting Enhancements

Enhancement suggestions are welcome! Please:

- **Use a clear and descriptive title**
- **Provide a detailed description** of the enhancement
- **Explain why this enhancement would be useful**
- **Include examples** of how it would work

### Pull Requests

We actively welcome your pull requests:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Test your changes thoroughly
5. Commit your changes (`git commit -m 'Add amazing feature'`)
6. Push to your branch (`git push origin feature/amazing-feature`)
7. Open a Pull Request

## Development Setup

### Prerequisites

- Git
- GitHub account with access to 1215-Labs organization
- Basic understanding of GitHub Actions

### Local Development

1. Clone the repository:
   ```bash
   git clone https://github.com/1215-Labs/.github.git
   cd .github
   ```

2. Create a feature branch:
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. Make your changes to the appropriate files:
   - Reusable workflows: `.github/workflows/reusable-*.yml`
   - Composite actions: `.github/actions/*/action.yml`
   - Workflow templates: `workflow-templates/`
   - Documentation: `*.md`

## Contribution Guidelines

### Reusable Workflows

When contributing reusable workflows:

1. **Use `workflow_call` trigger**
   ```yaml
   on:
     workflow_call:
       inputs:
         # Define inputs
       secrets:
         # Define secrets
   ```

2. **Document all inputs and secrets**
   - Provide clear descriptions
   - Specify if required or optional
   - Include default values where appropriate

3. **Set appropriate permissions**
   ```yaml
   permissions:
     contents: read
     # Only request permissions you need
   ```

4. **Include timeout settings**
   ```yaml
   timeout-minutes: 30  # Prevent stuck workflows
   ```

5. **Add concurrency controls**
   ```yaml
   concurrency:
     group: workflow-${{ github.ref }}
     cancel-in-progress: true
   ```

6. **Use latest stable action versions**
   - `actions/checkout@v4`
   - `actions/setup-node@v4`
   - etc.

### Composite Actions

When contributing composite actions:

1. **Include metadata**
   ```yaml
   name: 'Action Name'
   description: 'Clear description'
   author: '1215-Labs'
   branding:
     icon: 'icon-name'
     color: 'color-name'
   ```

2. **Define inputs with defaults**
   ```yaml
   inputs:
     input-name:
       description: 'Clear description'
       required: false
       default: 'default-value'
   ```

3. **Use `shell: bash` for all steps**
   ```yaml
   runs:
     using: 'composite'
     steps:
       - shell: bash
         run: |
           # Your commands
   ```

4. **Add error handling**
   ```bash
   if [ $? -ne 0 ]; then
     echo "❌ Error occurred"
     exit 1
   fi
   ```

### Workflow Templates

When contributing workflow templates:

1. **Create both `.yml` and `.properties.json` files**
   - `template-name.yml` - The workflow
   - `template-name.properties.json` - The metadata

2. **Include complete properties**
   ```json
   {
     "name": "Template Name",
     "description": "Detailed description",
     "iconName": "icon-name",
     "categories": ["Category1", "Category2"],
     "filePatterns": ["pattern1$", "pattern2$"]
   }
   ```

3. **Reference the organization's reusable workflows**
   ```yaml
   uses: 1215-Labs/.github/.github/workflows/reusable-*.yml@main
   ```

### Documentation

When contributing documentation:

1. **Keep it clear and concise**
2. **Include code examples**
3. **Update the table of contents**
4. **Add troubleshooting tips** when relevant
5. **Link to related documentation**

## Style Guidelines

### YAML Style

- Use 2 spaces for indentation
- Use single quotes for strings
- Add comments for complex logic
- Group related steps together
- Use descriptive step names

Example:
```yaml
- name: Install dependencies
  working-directory: ${{ inputs.working-directory }}
  run: |
    # Check for lock files and install accordingly
    if [ -f "package-lock.json" ]; then
      npm ci
    else
      npm install
    fi
```

### Markdown Style

- Use ATX-style headers (`#`, `##`, etc.)
- Add blank lines around code blocks
- Use tables for structured data
- Include examples where helpful
- Link to relevant resources

### Naming Conventions

- **Workflows**: `reusable-feature-name.yml`
- **Actions**: `action-name` (directory), `action.yml` (file)
- **Templates**: `category-purpose.yml`
- **Examples**: `example-scenario.yml`

## Testing Your Changes

### Test Reusable Workflows

1. Create a test repository (or use an existing one)
2. Create a workflow that calls your reusable workflow
3. Push and verify the workflow runs correctly
4. Check all success and failure paths

Example test workflow:
```yaml
name: Test Reusable Workflow

on: [push]

jobs:
  test:
    uses: YOUR-USERNAME/.github/.github/workflows/reusable-ci.yml@your-branch
    with:
      language: 'nodejs'
      node-version: '18'
```

### Test Composite Actions

1. Create a test workflow in a test repository
2. Reference your action using your fork and branch
3. Verify all inputs and outputs work correctly

Example:
```yaml
steps:
  - uses: YOUR-USERNAME/.github/.github/actions/setup-nodejs-environment@your-branch
    with:
      node-version: '18'
```

### Validation Checklist

Before submitting:

- [ ] Workflow syntax is valid (YAML linter)
- [ ] All inputs are documented
- [ ] Default values are reasonable
- [ ] Error handling is in place
- [ ] Timeouts are set appropriately
- [ ] Permissions are minimal
- [ ] Documentation is updated
- [ ] Examples are provided
- [ ] Tested in a real repository

## Submitting Changes

### Pull Request Process

1. **Update documentation**
   - Update `WORKFLOWS.md` for workflow changes
   - Update `README.md` if adding major features
   - Add/update examples in `examples/`

2. **Write a clear PR description**
   ```markdown
   ## Changes
   - Added new feature X
   - Fixed bug Y
   - Improved documentation for Z
   
   ## Testing
   - Tested in repository: owner/repo
   - Workflow runs: [link]
   
   ## Breaking Changes
   - None / List any breaking changes
   ```

3. **Link related issues**
   - Reference issues in the description
   - Use keywords like "Fixes #123"

4. **Request review**
   - Tag relevant team members
   - Be responsive to feedback

5. **Keep PR focused**
   - One feature/fix per PR
   - Avoid mixing unrelated changes

### Commit Message Guidelines

Use clear, descriptive commit messages:

```
Add reusable security scan workflow

- Implements CodeQL analysis
- Adds dependency scanning
- Includes secret detection
- Updates documentation
```

Format:
- Use imperative mood ("Add" not "Added")
- First line: brief summary (50 chars max)
- Blank line, then detailed description if needed
- Reference issues: "Fixes #123"

## Release Process

Maintainers will:

1. Review and test contributions
2. Merge approved PRs to `main`
3. Tag releases with semantic versioning
4. Update changelog
5. Announce updates to the organization

## Getting Help

- 📖 Read the [documentation](WORKFLOWS.md)
- 💬 [Start a discussion](https://github.com/1215-Labs/.github/discussions)
- 🐛 [Report issues](https://github.com/1215-Labs/.github/issues)
- 📧 Contact the DevOps team

## Recognition

Contributors will be recognized in:
- Release notes
- Repository contributors page
- Special acknowledgments for major contributions

Thank you for contributing to 1215-Labs! 🚀
