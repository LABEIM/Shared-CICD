# Shared CI/CD Framework

A production-grade, modular CI/CD framework for web applications. Built with a **Hybrid Architecture** combining:
1. **Composable Actions**: Independent building blocks for runtime setup, multi-cloud deployments, PR feedback, and performance audits.
2. **Reusable Workflows**: Complete out-of-the-box pipelines that downstream projects can adopt with a 15-line workflow file.

---

## Architecture Overview

```mermaid
flowchart TD
    subgraph SharedRepo ["Central Shared CI/CD Repository"]
        subgraph Actions ["Standalone Composite Actions"]
            A1["actions/setup-and-build\n(Auto Lockfiles + Monorepo + Framework Out-Dirs)"]
            A2["actions/deploy-cloudflare\n(Wrangler + retries + robust URL resolution)"]
            A3["actions/deploy-vercel\n(Vercel Output API v3 + fast npx + SHA filtering)"]
            A4["actions/pr-preview-comment\n(Dynamic preview table + anchor tags + safe perms)"]
            A5["actions/lighthouse-audit\n(Treosh LHCI + parsed scorecard summaries)"]
        end

        subgraph Workflows ["Master Reusable Workflows"]
            W1[".github/workflows/deploy-app.yml\n(Feature toggles: enable_cloudflare, enable_vercel, etc.)"]
            W2[".github/workflows/codeql-scan.yml\n(CodeQL SAST security scanning)"]
            W3[".github/workflows/dependency-review.yml\n(Dependency vulnerability gate)"]
            W4[".github/workflows/stale-cleanup.yml\n(Automated stale issues / PR lifecycle)"]
        end
    end

    A1 --> W1
    A2 --> W1
    A3 --> W1
    A4 --> W1
    A5 --> W1

    subgraph Downstream ["Consumer Repositories"]
        P1["Project A (Cloudflare Only)\nuses: .../deploy-app.yml@v1\nenable_cloudflare: true\nenable_vercel: false"]
        P2["Project B (Cloudflare + Vercel)\nuses: .../deploy-app.yml@v1\nenable_cloudflare: true\nenable_vercel: true"]
        P3["Project C (Monorepo App)\nworking_directory: apps/web"]
        P4["Project D (Custom Pipeline)\nuses individual composite actions"]
    end

    W1 --> P1
    W1 --> P2
    W1 --> P3
    A2 -.-> P4
```

---

## Quickstart

### 1. Pure Static / Zero-Dependency Site (Vanilla HTML/CSS/JS)
**Zero configuration required!** No `package.json` or build scripts needed. The workflow auto-detects static repositories, bypasses dependency installation and build steps, stages static assets, and deploys in **< 15 seconds**:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read
  pull-requests: write
  deployments: write

jobs:
  deploy:
    uses: my-org/shared-ci-cd/.github/workflows/deploy-app.yml@v1
    with:
      enable_cloudflare: true
      cloudflare_project_name: 'my-static-site'
    secrets: inherit
```

---

### 2. Standard Framework Site (Auto-Detected Package Manager & Output)
In your project repository, create `.github/workflows/ci-cd.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read
  pull-requests: write
  deployments: write

jobs:
  deploy:
    uses: LABEIM/shared-ci-cd/.github/workflows/deploy-app.yml@v1
    with:
      node_version: '22'
      package_manager: 'auto' # Auto-detects npm, pnpm, bun, or yarn from lockfile
      dist_dir: 'auto'        # Auto-detects dist, out, build, .output/public, etc.
      enable_cloudflare: true
      cloudflare_project_name: 'my-web-app'
      enable_vercel: false
      lighthouse_routes: |
        /
        /about
        /contact
    secrets: inherit
```

---

### 3. Monorepo / Subdirectory App (Turborepo, Nx, pnpm workspaces)
Deploy an application located in a subfolder (e.g. `apps/web`):

```yaml
jobs:
  deploy:
    uses: my-org/shared-ci-cd/.github/workflows/deploy-app.yml@v1
    with:
      working_directory: 'apps/web'
      package_manager: 'auto'
      dist_dir: 'auto'
      enable_cloudflare: true
      cloudflare_project_name: 'monorepo-web-app'
    secrets: inherit
```

---

### 4. Multi-Cloud Parallel Deployment (Cloudflare + Vercel Backup)
```yaml
jobs:
  deploy:
    uses: LABEIM/shared-ci-cd/.github/workflows/deploy-app.yml@v1
    with:
      node_version: '22'
      package_manager: 'auto'
      dist_dir: 'auto'

      # Both platforms deployed in parallel
      enable_cloudflare: true
      cloudflare_project_name: 'production-store'

      enable_vercel: true
      vercel_project_id: ${{ vars.VERCEL_PROJECT_ID }}
      vercel_org_id: ${{ vars.VERCEL_ORG_ID }}

      lighthouse_routes: |
        /
        /pricing
    secrets: inherit
```

---

### 5. Using Standalone Composite Actions Directly
If your project has custom Docker builds, staging environments, or multi-step release gates, import composite actions directly:

```yaml
jobs:
  custom-build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build site
        uses: LABEIM/shared-ci-cd/actions/setup-and-build@v1
        with:
          package_manager: 'auto'
          build_command: 'npm run build'
          dist_dir: 'auto'

      - name: Deploy to Cloudflare
        id: cf
        uses: LABEIM/shared-ci-cd/actions/deploy-cloudflare@v1
        with:
          project_name: 'my-custom-app'
          dist_dir: 'dist'
          api_token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          account_id: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}

      - name: Post PR Comment
        if: github.event_name == 'pull_request'
        uses: LABEIM/shared-ci-cd/actions/pr-preview-comment@v1
        with:
          cloudflare_url: ${{ steps.cf.outputs.deploy_url }}
```

---

## Configuration Reference

### Reusable Workflow (`deploy-app.yml`)

#### Inputs (`with:`)

| Input | Type | Default | Description |
| :--- | :---: | :---: | :--- |
| `working_directory` | String | `'.'` | Working directory for monorepos or subfolder apps (e.g. `apps/web`) |
| `node_version` | String | `'22'` | Node.js version (or `'auto'` to read `.nvmrc` / `.node-version`) |
| `package_manager` | String | `'auto'` | Package manager (`'auto'`, `'npm'`, `'pnpm'`, `'bun'`, `'yarn'`, `'none'`, `'static'`) |
| `install_command` | String | `''` | Custom install command override (or `'none'` to skip) |
| `check_command` | String | `''` | Optional diagnostic check (e.g. `npx astro check`, `npm run lint`) |
| `build_command` | String | `'npm run build'` | Static build command (or `'none'` for static/pre-built sites, or `'auto'`) |
| `dist_dir` | String | `'dist'` | Static output directory (`'dist'`, `'out'`, `'build'`, `'.output/public'`, `'auto'`) |
| `enable_cloudflare` | Boolean | `true` | Enable Cloudflare Pages deployment |
| `cloudflare_project_name` | String | `''` | Cloudflare Pages project name |
| `production_branch` | String | `'main'` | Production branch name |
| `enable_vercel` | Boolean | `false` | Enable Vercel parallel/backup deployment |
| `vercel_project_id` | String | `''` | Vercel Project ID |
| `vercel_org_id` | String | `''` | Vercel Organization ID or Team ID |
| `enable_pr_comment` | Boolean | `true` | Automatically post/update PR preview comments |
| `enable_lighthouse` | Boolean | `true` | Run automated Lighthouse CI performance audit |
| `lighthouse_routes` | String | `'/'` | Space or newline-separated list of relative paths |
| `lighthouse_config_path` | String | `''` | Optional custom `.lighthouserc.json` path |

#### Secrets (`secrets:`)

| Secret | Required | Description |
| :--- | :---: | :--- |
| `CLOUDFLARE_API_TOKEN` | If CF enabled | Cloudflare API token with `Account.Cloudflare Pages:Edit` permissions |
| `CLOUDFLARE_ACCOUNT_ID` | If CF enabled | Cloudflare Account ID (32-character hexadecimal string) |
| `VERCEL_TOKEN` | If Vercel enabled | Vercel personal access token or automation token |

---

## Organization & Repository Setup

### 1. Configure Organization Secrets
Store shared infrastructure tokens once at the GitHub Organization level:
* Go to **Organization Settings** -> **Secrets and variables** -> **Actions**.
* Add:
  - `CLOUDFLARE_API_TOKEN`
  - `CLOUDFLARE_ACCOUNT_ID`
  - `VERCEL_TOKEN`
* Set **Repository access** to `Selected repositories` or `All repositories`.

### 2. Enable Workflow Access (Private/Internal Repositories)
To allow other repositories in your GitHub Organization to call this workflow:
1. In this shared repository, navigate to **Settings** -> **Actions** -> **General**.
2. Under **Access**, select **Accessible from repositories in the '[your-org]' organization**.

### 3. Release & Versioning Strategy
* Consumer projects should reference major version tags (e.g. `@v1`):
  ```yaml
  uses: LABEIM/shared-ci-cd/.github/workflows/deploy-app.yml@v1
  ```
* For detailed step-by-step tag management and release instructions, see [RELEASING.md](RELEASING.md).

---

## Framework Recipes

### Pure Static / Vanilla HTML, CSS & JS (Zero Configuration)
* Automatically skips package management & builds when `package.json` is missing.
* Stages static root files or `public/` folder directly.
```yaml
with:
  enable_cloudflare: true
  cloudflare_project_name: 'my-static-site'
```

### Astro
```yaml
with:
  package_manager: 'auto'
  check_command: 'npx astro check'
  build_command: 'npm run build'
  dist_dir: 'dist'
```

### Next.js (Static Export)
* Ensure `next.config.js` contains `output: 'export'`
```yaml
with:
  package_manager: 'auto'
  build_command: 'npm run build'
  dist_dir: 'out' # or 'auto'
```

### Nuxt (Static / SSG)
```yaml
with:
  package_manager: 'auto'
  build_command: 'npx nuxi generate'
  dist_dir: '.output/public' # or 'auto'
```

### Vite (React / Vue / Svelte)
```yaml
with:
  package_manager: 'auto'
  build_command: 'npm run build'
  dist_dir: 'dist'
```

### Hugo / Jekyll / MkDocs
```yaml
with:
  package_manager: 'none'
  dist_dir: 'public' # or 'site'
```

---

## Auxiliary Reusable Workflows

In addition to web deployment, this repository provides:
* [`.github/workflows/codeql-scan.yml`](.github/workflows/codeql-scan.yml) - Static security analysis for JavaScript, TypeScript, Python, and Go.
* [`.github/workflows/dependency-review.yml`](.github/workflows/dependency-review.yml) - High-severity supply chain vulnerability gate.
* [`.github/workflows/stale-cleanup.yml`](.github/workflows/stale-cleanup.yml) - Automated issue and PR cleanup.
* [`.github/workflows/dependabot-auto-merge.yml`](.github/workflows/dependabot-auto-merge.yml) - Auto-merging safe Dependabot updates.
