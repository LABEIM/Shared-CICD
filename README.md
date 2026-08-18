# Shared CI/CD Framework (Approach C)

A production-grade, modular CI/CD framework for web applications. Built with a **Hybrid Architecture** combining:
1. **Composable Actions**: Independent building blocks for runtime setup, multi-cloud deployments, PR feedback, and performance audits.
2. **Reusable Workflows**: Complete out-of-the-box pipelines that downstream projects can adopt with a 15-line workflow file.

---

## Architecture Overview

```mermaid
flowchart TD
    subgraph SharedRepo ["Central Shared CI/CD Repository"]
        subgraph Actions ["Standalone Composite Actions"]
            A1["actions/setup-and-build\n(npm / pnpm / bun / yarn + cache)"]
            A2["actions/deploy-cloudflare\n(Wrangler + retries + secret-safe URL resolution)"]
            A3["actions/deploy-vercel\n(Vercel Output API v3 + prebuilt + retries)"]
            A4["actions/pr-preview-comment\n(Dynamic preview table + deduplication)"]
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
        P3["Project C (Custom Pipeline)\nuses individual composite actions"]
    end

    W1 --> P1
    W1 --> P2
    A2 -.-> P3
```

---

## Quickstart

### 1. Cloudflare Pages (Default & Recommended)
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
    uses: my-org/shared-ci-cd/.github/workflows/deploy-app.yml@v1
    with:
      node_version: '22'
      package_manager: 'npm' # npm | pnpm | bun | yarn
      build_command: 'npm run build'
      dist_dir: 'dist'
      enable_cloudflare: true
      cloudflare_project_name: 'my-web-app'
      enable_vercel: false # Vercel is disabled & uses 0 runner minutes
      lighthouse_routes: |
        /
        /about
        /contact
    secrets: inherit
```

---

### 2. Multi-Cloud Parallel Deployment (Cloudflare + Vercel Backup)
```yaml
jobs:
  deploy:
    uses: my-org/shared-ci-cd/.github/workflows/deploy-app.yml@v1
    with:
      node_version: '22'
      package_manager: 'pnpm'
      build_command: 'pnpm build'
      dist_dir: 'dist'

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

### 3. Using Standalone Composite Actions Directly
If your project has custom Docker builds, staging environments, or multi-step release gates, import composite actions directly:

```yaml
jobs:
  custom-build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build site
        uses: my-org/shared-ci-cd/actions/setup-and-build@v1
        with:
          package_manager: 'bun'
          build_command: 'bun run build'
          dist_dir: 'out'

      - name: Deploy to Cloudflare
        id: cf
        uses: my-org/shared-ci-cd/actions/deploy-cloudflare@v1
        with:
          project_name: 'my-custom-app'
          dist_dir: 'out'
          api_token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          account_id: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}

      - name: Post PR Comment
        if: github.event_name == 'pull_request'
        uses: my-org/shared-ci-cd/actions/pr-preview-comment@v1
        with:
          cloudflare_url: ${{ steps.cf.outputs.deploy_url }}
```

---

## Configuration Reference

### Reusable Workflow (`deploy-app.yml`)

#### Inputs (`with:`)

| Input | Type | Default | Description |
| :--- | :---: | :---: | :--- |
| `node_version` | String | `'22'` | Node.js version |
| `package_manager` | String | `'npm'` | Package manager (`npm`, `pnpm`, `bun`, `yarn`) |
| `install_command` | String | `''` | Custom install command override |
| `check_command` | String | `''` | Optional diagnostic check (e.g. `npx astro check`, `npm run lint`) |
| `build_command` | String | `'npm run build'` | Static build command |
| `dist_dir` | String | `'dist'` | Static export directory to deploy |
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
  uses: my-org/shared-ci-cd/.github/workflows/deploy-app.yml@v1
  ```
* When publishing non-breaking updates or patches, update the floating `@v1` git tag:
  ```bash
  git tag -fa v1 -m "Release v1.2.0"
  git push origin v1 --force
  ```
* Introduce `@v2` only when making breaking changes (e.g. changing mandatory inputs).

---

## Framework Recipes

### Astro
```yaml
with:
  package_manager: 'npm'
  check_command: 'npx astro check'
  build_command: 'npm run build'
  dist_dir: 'dist'
```

### Next.js (Static Export)
* Ensure `next.config.js` contains `output: 'export'`
```yaml
with:
  package_manager: 'pnpm'
  build_command: 'pnpm build'
  dist_dir: 'out'
```

### Vite (React / Vue / Svelte)
```yaml
with:
  package_manager: 'bun'
  build_command: 'bun run build'
  dist_dir: 'dist'
```

---

## Auxiliary Reusable Workflows

In addition to web deployment, this repository provides:
* [`.github/workflows/codeql-scan.yml`](.github/workflows/codeql-scan.yml) - Static security analysis for JavaScript, TypeScript, Python, and Go.
* [`.github/workflows/dependency-review.yml`](.github/workflows/dependency-review.yml) - High-severity supply chain vulnerability gate.
* [`.github/workflows/stale-cleanup.yml`](.github/workflows/stale-cleanup.yml) - Automated issue and PR cleanup.
