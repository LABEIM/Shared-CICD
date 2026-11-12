# AGENTS.md

## Section 1: Agent Operational Workflow

### 1.1 Startup Workflow
Before writing any code, the agent must complete these steps:
1. Confirm the working directory using `pwd`.
2. Inspect the repository structure and key configuration files (`.github/workflows/`, `actions/`, `examples/`, `README.md`, `RELEASING.md`) to understand the current architecture and state.
3. Review recent repository history by checking the last 5 commits using `git log --oneline -5` and the current release tags using `git tag --sort=-version:refname`.
4. Run baseline verification using `actionlint` before starting new work. If baseline verification fails, fix that issue first before stacking new feature work on top of a broken starting state.

### 1.2 Working Rules
1. Focus entirely on one feature or task at a time.
2. Do not mark a task or feature complete just because code was added. Ensure the YAML parses, `actionlint` passes, and examples remain valid.
3. Adhere to `.editorconfig` conventions (UTF-8, LF, 2-space indentation) and keep YAML/style consistent with the existing workflows and actions.
4. Keep all changes within the selected task scope unless a critical blocker forces a narrow supporting fix.
5. Do not silently alter verification rules or relax validation during implementation.
6. Prefer durable repository artifacts and updated documentation over transient chat summaries.
7. When changing action inputs/outputs, workflow inputs/secrets, dependency versions, or the release process, update the corresponding documentation (`README.md`, `RELEASING.md`, `CONTRIBUTING.md`) and the affected `examples/*.yml` caller files to reflect the new contract.

### 1.3 Required Artifacts & Key Files
1. `.github/workflows/deploy-app.yml`: Main reusable deployment workflow and its input/secret contract.
2. `actions/*/action.yml`: Composite action definitions for setup/build, Cloudflare, Vercel, PR comments, and Lighthouse.
3. `.github/workflows/lint-ci.yml`: Repository validation pipeline (`actionlint` plus YAML syntax checks).
4. `examples/*.yml`: Canonical consumer workflow examples that must stay in sync with the workflow inputs.
5. `README.md` and `RELEASING.md`: Usage reference and SemVer/floating-tag release strategy.

### 1.4 Definition Of Done
A feature achieves the status of completed only when all of the following conditions are met:
1. The target behavior is fully implemented and satisfies requirements.
2. All YAML files are syntactically valid.
3. `actionlint` reports zero errors on `.github/workflows/`, and every `actions/*/action.yml` file parses as valid YAML.
4. `examples/*.yml` are updated and reference only inputs/secrets that exist in the reusable workflow.
5. The `.github/workflows/lint-ci.yml` validation pipeline passes on the branch/PR.
6. No credentials, tokens, or private values are committed; consumer credentials are passed through `secrets: inherit` or organization-level secrets.

### 1.5 End Of Session Workflow
Before ending a session, the agent must perform these tasks:
1. Confirm no trailing lint or YAML validation errors remain by running the validation commands.
2. Explicitly record any unresolved risk, technical debt, or workflow blocker in the session handoff.
3. Commit the changes with a descriptive message adhering to the Conventional Commits specification once the work is in a safe state.

---

## Section 2: Tech Stack & Architecture

### 2.1 Framework & Core Stack
- **Platform**: GitHub Actions (composite actions and reusable workflows)
- **Configuration Language**: YAML (GitHub Actions syntax)
- **Scripting**: Bash (`shell: bash`) with `shellcheck`-clean shell code
- **Validation Tooling**: `actionlint` (via `reviewdog/action-actionlint`), Node.js for YAML syntax checks
- **Deployment Targets**: Cloudflare Workers (Wrangler direct deploy or Deploy Hooks), Vercel (Build Output API v3 or remote cloud builds)
- **Security & Supply Chain**: CodeQL, dependency review, Dependabot, GitHub secret scanning
- **Versioning**: Semantic Versioning with floating major tags (`v1`, `v2`, ...)
- **Runtime & Package Manager**: Node.js `>=22.12` for the Cloudflare deploy job, `>=20` for validation scripts; no application package manager or lockfile in this repository

### 2.2 Directory Layout & Component Roles
- `actions/setup-and-build/`: Detects package manager and output directory, installs dependencies, runs checks, builds the site, and uploads the dist artifact.
- `actions/deploy-cloudflare/`: Deploys static assets or a Worker + assets (Astro 7 SSR) to Cloudflare Workers via Wrangler, or triggers remote Workers Builds via Deploy Hooks, with retries and URL resolution.
- `actions/deploy-vercel/`: Deploys prebuilt output via the Build Output API v3 or delegates the build to Vercel remote infrastructure.
- `actions/pr-preview-comment/`: Posts or updates a single PR comment with live preview links.
- `actions/lighthouse-audit/`: Runs Lighthouse CI audits and produces parsed scorecard summaries.
- `.github/workflows/`: Reusable and repository-local workflows (deployment, CodeQL, dependency review, stale cleanup, Dependabot auto-merge, release tagging, linting).
- `examples/`: Self-contained caller workflow examples for static sites, frameworks, monorepos, multi-cloud, and remote-build scenarios.

### 2.3 Role-Based Access Control & Security Scoping
- **Authentication**: No end-user authentication. Workflows authenticate to GitHub, Cloudflare, and Vercel using consumer-supplied tokens.
- **Authorization / RBAC**: Not applicable; access is governed by GitHub repository permissions and organization secret scoping.
- **Protected Routes**: Not applicable.
- **Required Permissions**: Callers should request only `contents: read`, `pull-requests: write`, and `deployments: write` unless a workflow explicitly needs more.

### 2.4 Service & Business Logic Layer
- **Workflow Logic**: Implemented as Bash steps inside composite actions and reusable workflows; outputs flow through `$GITHUB_OUTPUT` and job outputs.
- **External Services & Integrations**: GitHub REST API, Cloudflare API, Vercel API, Workers Builds Deploy Hooks, and Lighthouse CI (`@lhci/cli`).

---

## Section 3: Privacy, Security & Specifications

### 3.1 Privacy & Environment Safety
1. Do not hardcode personal user data, private keys, or credentials in source code, comments, or documentation. Keep contact references public and generic.
2. Secrets management: Never commit active environment secrets. Keep `.env`, `.env.local`, and credential files excluded via `.gitignore`. Document required secrets in `README.md`.

### 3.2 Security Rules
1. **Input Sanitization & Injection Prevention**: Treat all workflow and action inputs as untrusted. Pass values through `env:` and quote them; never interpolate `${{ inputs.* }}` directly into a shell script where injection is possible. Sanitize URLs and identifiers (as `pr-preview-comment` does) before rendering them.
2. **Secret vs variable storage**: real credentials (`CLOUDFLARE_API_TOKEN`, `VERCEL_TOKEN`, `CLOUDFLARE_DEPLOY_HOOK_URL`) are GitHub secrets. Identifiers (`VERCEL_PROJECT_ID`, `VERCEL_ORG_ID`, `CLOUDFLARE_ACCOUNT_ID` when passed via `cloudflare_account_id`) and public build config (`PUBLIC_*`) should be stored as variables; `PUBLIC_*` secrets remain supported, and a same-named non-empty variable overrides them unless the values differ, which fails the build. Runtime credentials (for example a CMS API token consumed by an SSR Worker) live in the platform's secret store, never in GitHub; the Keystatic Cloud path in `USAGE.md` needs none. Environment secrets require a public repository on GitHub Free; private repositories on Free must use repository secrets.
3. **Least Privilege**: Declare the minimal `permissions:` block for every workflow. Do not grant `contents: write` unless tagging or releasing.
4. **Credential Handling**: Never echo tokens. Validate required credentials at the start of a deployment step and fail with `::error::` when they are missing.
5. **Third-Party Actions**: Pin actions to a major version tag (`@v4`) and review changes before bumping major versions.

### 3.3 Data & State
- No database or persistent state. Deployment state, artifacts, and PR comments are managed through GitHub Actions artifacts and the Cloudflare/Vercel APIs.

---

## Section 4: Development, Testing & Operations

### 4.1 Best Coding Practices & Standards
1. **GitHub Actions & YAML**:
   - Validate every workflow and action with `actionlint`.
   - Declare `description`, `required`, and `default` for every action input; document outputs with their `value` expressions.
   - Use `$GITHUB_OUTPUT` and `$GITHUB_ENV` for step communication instead of deprecated `::set-output` commands.
   - Set `timeout-minutes` on jobs and use `concurrency` to cancel superseded runs.
2. **Bash & Shell Code**:
   - Quote variables, prefer `set -euo pipefail` where compatible, and avoid parsing human-readable CLI output when a machine-readable API is available.
   - Keep retry/backoff logic explicit and bounded (`max_retries`, `retry_delay`).
   - Ensure shell code is `shellcheck`-clean.
3. **Determinism & Idempotency**:
   - PR comments must update an existing comment rather than creating duplicates.
   - Deployments should resolve deterministic URLs and report them through outputs.
4. **Documentation & Contracts**:
   - Any new input, secret, or output must be documented in `README.md` and exercised in `examples/`.
   - Versioning and tag behavior must be reflected in `RELEASING.md`.
5. **Formatting & Commit Conventions**:
   - Indentation: 2 spaces for YAML, JSON, and Markdown code fences.
   - Commit conventions: All commits must follow the [Conventional Commits](https://www.conventionalcommits.org/) specification (`feat:`, `fix:`, `docs:`, `style:`, `refactor:`, `test:`, `chore:`).

### 4.2 Testing & Portability
1. **Verification Commands**:
   - Lint workflows: `actionlint`
   - Validate composite action metadata as YAML (covered by the `lint-ci.yml` syntax check)
   - Validate YAML syntax: run the Ruby/Psych validation step from `.github/workflows/lint-ci.yml`
   - Build: not applicable — this repository ships YAML definitions and has no compilation step
2. **Test Isolation**: Validate workflow changes on a branch or fork before merging. Use `workflow_dispatch` and example caller workflows to exercise changes without affecting consumers.
3. **Consumer Impact**: Because consumers pin a floating major tag (e.g. `@v2`), non-breaking changes must preserve the existing input/secret contract.

### 4.3 Execution Environment
- **Runners**: GitHub-hosted `ubuntu-latest`.
- **Ports / Dev Server**: Not applicable — no long-running service or local dev server.
