# AI Workflow Setup (Claude / Codex / Cursor) — reusable across repos

These workflows are triggered by commenting special commands like `@claude-fix` on an issue/PR. To reuse them in other repos, copy `.github/workflows/*.yml` plus the prompt templates in `.github/` and set the variables/secrets below.

## 1) Authorization (who can trigger by commenting)

The workflows gate execution using a JSON allowlist.

- Create a **repository variable** named `AI_WORKFLOW_AUTHORIZED_USERS_JSON`
  - Location: **Repo Settings → Secrets and variables → Actions → Variables**
  - Value must be valid JSON array of GitHub usernames, e.g.:
    - `["mdc159"]`
    - `["mdc159","teammate1"]`

If you do **not** set this variable, the workflows default to `["mdc159"]`.

## 2) Required secrets (API credentials)

Each repo needs secrets for whichever workflows you plan to use:

- **Claude workflows (`claude-*.yml`)**
  - `CLAUDE_CODE_OAUTH_TOKEN`
- **Codex workflows (`codex-*.yml`)**
  - `OPENAI_API_KEY`
- **Cursor workflows (`cursor-*.yml`)**
  - `CURSOR_API_KEY`

Location: **Repo Settings → Secrets and variables → Actions → Secrets**

> Note: `GITHUB_TOKEN` is provided automatically by GitHub Actions; you do not create it.

## 3) “Use my GitHub credentials” clarification

These workflows authenticate to GitHub using GitHub Actions tokens:

- For reading/writing repo content and posting comments/PRs, they use **`GITHUB_TOKEN`** by default.
  - That token acts as `github-actions[bot]` within the repository.
  - Permissions are limited by the workflow’s `permissions:` block.

If you need changes to be authored by **your user account**, you can switch specific steps/actions to use a **PAT**:

- Create a secret like `GH_PAT` (fine-grained PAT recommended)
- Then update steps that use `secrets.GITHUB_TOKEN` to use `secrets.GH_PAT`

## 4) Copy/paste checklist for a new repo

- Copy these files:
  - `.github/workflows/*.yml`
  - `.github/issue_fix_prompt.md`
  - `.github/pr_review_prompt.md`
- Set `AI_WORKFLOW_AUTHORIZED_USERS_JSON`
- Add the secrets you need (`CLAUDE_CODE_OAUTH_TOKEN` / `OPENAI_API_KEY` / `CURSOR_API_KEY`)

## 5) Scaling to “all my repos” (recommended)

If your repos are under a GitHub **Organization**, you can define **organization-level** secrets/variables once and expose them to many repos (with allowlists), so you don’t have to copy secrets repo-by-repo.

### Organization-level secrets & variables (one-time org setup)

You can centrally manage credentials and configuration for these workflows by creating **organization-level**:

- **Secrets** (for API keys / tokens)
- **Variables** (for non-sensitive config like allowlists)

This lets every opted-in repository reference the same names via `secrets.<NAME>` / `vars.<NAME>`, without duplicating values per repo.

Per GitHub’s “Using secrets in GitHub Actions” docs, org secrets can be configured with access policies, and secrets have important limitations (for example, secrets generally won’t be provided to workflows triggered from forks, and secrets are not automatically passed into reusable workflows) — see [Using secrets in GitHub Actions](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets).

#### 1) Create organization-level secrets

Create these in your organization:

- `CLAUDE_CODE_OAUTH_TOKEN` (if you use `claude-*.yml`)
- `OPENAI_API_KEY` (if you use `codex-*.yml`)
- `CURSOR_API_KEY` (if you use `cursor-*.yml`)

Then **scope access** to either:

- **All repositories** (easy, broader blast radius)
- **Selected repositories** (recommended)

Tip: keep the secret names identical to what the workflows expect so you don’t have to edit any YAML.

#### 2) Create organization-level variables

Create an organization variable:

- `AI_WORKFLOW_AUTHORIZED_USERS_JSON`

Then scope it the same way (all vs selected repos). Repo-level variables (if present) will override org-level variables, so you can make exceptions per repository when needed.

### Using org-level secrets/variables in these workflows

No YAML changes are required as long as:

- The org-level secret/variable **names match** what the workflow references, and
- The repository is included in the org secret/variable’s **access policy**

The workflow can keep referencing:

```yaml
env:
  CURSOR_API_KEY: ${{ secrets.CURSOR_API_KEY }}
  AI_WORKFLOW_AUTHORIZED_USERS_JSON: ${{ vars.AI_WORKFLOW_AUTHORIZED_USERS_JSON }}
```

### Reusable workflows: centralize the logic once, call it from many repos

If you want to avoid copying `.github/workflows/*.yml` into every repo, you can put the “real” jobs into a single **workflow repo** (for example, `.github` or `ai-workflows`) and expose them as **reusable workflows** using `workflow_call`.

Important caveat from GitHub: **secrets are not automatically passed to reusable workflows** — the caller must explicitly pass them (or you must otherwise structure your workflow to not rely on implicit secret inheritance). See [Using secrets in GitHub Actions](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets).

#### Pattern A (recommended): call a reusable workflow and explicitly pass secrets

In the central repo (the reusable workflow), define inputs/secrets:

```yaml
on:
  workflow_call:
    secrets:
      CURSOR_API_KEY:
        required: true
    inputs:
      authorized_users_json:
        required: false
        type: string
```

In each target repo, keep a tiny “shim” workflow that calls the reusable one and passes org-level values:

```yaml
jobs:
  call-ai-workflow:
    uses: your-org/ai-workflows/.github/workflows/cursor-fix.yml@main
    secrets:
      CURSOR_API_KEY: ${{ secrets.CURSOR_API_KEY }}
    with:
      authorized_users_json: ${{ vars.AI_WORKFLOW_AUTHORIZED_USERS_JSON }}
```

This keeps secrets flow explicit and auditable while still letting you manage the actual workflow logic in one place.


