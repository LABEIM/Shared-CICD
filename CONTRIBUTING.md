# Contributing Guidelines

Thank you for your interest in contributing to **Shared CI/CD Framework**, a project under the **Enterprise Infrastructure Management (EIM) Research Laboratory**, Telkom University.

We welcome contributions from researchers, software engineers, students, and open-source enthusiasts. This document provides clear guidelines to ensure a collaborative, transparent, and efficient workflow.

---

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Intellectual Property & Licensing](#intellectual-property--licensing)
- [How Can I Contribute?](#how-can-i-contribute)
  - [Reporting Issues & Bugs](#reporting-issues--bugs)
  - [Requesting New Features](#requesting-new-features)
  - [Improving Documentation](#improving-documentation)
  - [Submitting Code Changes](#submitting-code-changes)
- [Getting Started & Local Setup](#getting-started--local-setup)
  - [Prerequisites](#prerequisites)
  - [Installation & Validation](#installation--validation)
- [Git & Contribution Workflow](#git--contribution-workflow)
  - [Branch Naming Standard](#branch-naming-standard)
  - [Commit Message Conventions](#commit-message-conventions)
  - [Commit Signing](#commit-signing)
- [Pull Request (PR) Lifecycle](#pull-request-pr-lifecycle)
  - [PR Submission Checklist](#pr-submission-checklist)
  - [CI/CD Verification](#cicd-verification)
  - [Code Review Process](#code-review-process)
- [Coding Standards & Quality Assurance](#coding-standards--quality-assurance)
- [Security Vulnerability Disclosures](#security-vulnerability-disclosures)
- [Community & Contact](#community--contact)

---

## Code of Conduct

All members, contributors, and maintainers are expected to uphold our [Code of Conduct](CODE_OF_CONDUCT.md). Please ensure you have read and understood our community standards before participating.

---

## Intellectual Property & Licensing

This project is open-source under the [MIT License](LICENSE). By contributing to this repository, you agree that your contributions will be licensed under the same MIT License terms that govern the project.

---

## How Can I Contribute?

### Reporting Issues & Bugs

If you discover an issue, defect, or unexpected behavior:
1. Check the [Issue Tracker](https://github.com/LABEIM/Shared-CICD/issues) to ensure the bug hasn't already been reported.
2. If unresolved, create a new issue using a concise, descriptive title.
3. Provide essential details:
   - **Description**: A clear summary of the unexpected behavior.
   - **Reproduction Steps**: A minimal caller workflow and the exact inputs/secrets used.
   - **Expected Behavior**: What you expected to happen.
   - **Environment**: Runner image, action/workflow version tag, and target platform (Cloudflare, Vercel, or both).
   - **Logs / Screenshots**: Redacted workflow logs, action outputs, or deployment errors.

### Requesting New Features

We welcome proposals for new features and optimizations:
1. Open a feature request in the issue tracker.
2. Describe the problem your proposal solves, target use cases, and suggested implementation approaches.
3. Discuss contract changes (new inputs, outputs, or secrets) with maintainers before opening large Pull Requests.

### Improving Documentation

High-quality documentation is vital for shared infrastructure:
- Clarifying usage examples, input/secret references, and architecture guides is always encouraged.
- Fix typos, broken links, or ambiguous explanations.

### Submitting Code Changes

Whether you are fixing a bug or adding a new feature, follow the git workflow detailed below.

---

## Getting Started & Local Setup

### Prerequisites

Ensure you have the required tools installed on your development machine:
- **Git** (latest stable)
- **actionlint** for workflow validation (`brew install actionlint` or `go install github.com/rhysd/actionlint/cmd/actionlint@latest`)
- **Node.js >= 20** for the YAML syntax validation step
- **Editor**: VS Code, Cursor, or your preferred IDE with YAML support

### Installation & Validation

1. **Fork and clone the repository:**
   ```bash
   git clone https://github.com/LABEIM/Shared-CICD.git
   cd shared-ci-cd
   ```

2. **Validate workflows:**
   ```bash
   actionlint
   ```

   Composite action metadata is validated as YAML by the CI syntax check.

3. **Verify YAML syntax:** the same check used by CI runs automatically in [`.github/workflows/lint-ci.yml`](.github/workflows/lint-ci.yml). You can run it through your editor's YAML parser or the Node.js script in that workflow.

> [!NOTE]
> `deploy-app.yml` references this repository's composite actions with absolute
> `LABEIM/shared-ci-cd/actions/<name>@v1` refs. A reusable workflow called from
> another repository cannot use `./actions/*` (that resolves against the caller's
> checkout), and `uses:` is static — expressions are not allowed. To test action
> changes before a release, exercise them in this repository with
> `uses: ./actions/<name>`, or temporarily repoint the workflow's refs to your
> branch and use a canary consumer (see [RELEASING.md](RELEASING.md)).

There are no application dependencies to install, no build step, and no local dev server.

---

## Git & Contribution Workflow

### Branch Naming Standard

Always create a new feature or fix branch from `main`. Do not commit directly to `main`.

```text
feat/add-bun-lockfile-detection
fix/cloudflare-retry-backoff
docs/update-workflow-inputs
perf/reduce-lighthouse-runtime
refactor/modularize-deploy-step
chore/bump-action-dependencies
```

**Common Branch Prefixes:**
| Prefix | Description |
| :--- | :--- |
| `feat/` | New action, workflow input, or functional capability |
| `fix/` | Bug fix or issue resolution |
| `docs/` | Documentation additions or adjustments |
| `perf/` | Performance optimizations |
| `refactor/` | YAML/shell restructuring with no behavior change |
| `test/` | Adding or updating validation coverage |
| `chore/` | Tooling, dependencies, or repository maintenance |

### Commit Message Conventions

We strictly enforce the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```text
<type>(<optional scope>): <imperative description>

[optional body explaining WHY the change was made]

[optional footer(s), e.g. Closes #123]
```

**Valid commit types:** `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`.

**Examples:**
```text
feat(setup-and-build): support bun lockfile detection
fix(deploy-cloudflare): handle missing deploy hook gracefully
docs(readme): document remote build inputs
ci(lint): pin actionlint to a fixed version
```

### Commit Signing

Signed commits using GPG or SSH keys are strongly encouraged across all EIM Lab repositories to verify authorship:

```bash
git commit -S -m "feat(lighthouse-audit): parse category scores"
```

---

## Pull Request (PR) Lifecycle

### PR Submission Checklist

Before submitting your Pull Request, ensure:
- [ ] `actionlint` passes with zero errors and all YAML files parse.
- [ ] All modified YAML files parse correctly.
- [ ] New or changed inputs, outputs, and secrets are documented in `README.md`.
- [ ] Affected `examples/*.yml` caller files are updated and remain valid.
- [ ] Documentation (`README.md`, `RELEASING.md`) reflects the change.
- [ ] Commit history is clean, signed, and adheres to Conventional Commits.
- [ ] Branch is rebased onto the latest `main` branch with no merge conflicts.

### CI/CD Verification

Once your PR is opened:
1. [`lint-ci.yml`](.github/workflows/lint-ci.yml) runs `actionlint` and YAML syntax validation.
2. [`codeql-scan.yml`](.github/workflows/codeql-scan.yml) and [`dependency-review.yml`](.github/workflows/dependency-review.yml) run security and supply-chain checks.
3. Ensure all CI checks pass green before requesting maintainer review.

### Code Review Process

- Project maintainers will review your PR for contract compatibility, shell safety, idempotency, and documentation.
- Maintainers may request modifications or suggest improvements.
- Once approved, your PR will be merged into `main`.
- Breaking contract changes require a new major version and must follow [`RELEASING.md`](RELEASING.md).

---

## Coding Standards & Quality Assurance

- **YAML Formatting:** Follow [`.editorconfig`](.editorconfig) (UTF-8, LF line endings, 2-space indentation). Keep keys ordered and expressions quoted where needed.
- **Action Contracts:** Every input must declare `description`, `required`, and a `default` where sensible. Every output must resolve to a real step output.
- **Shell Safety:** Quote variables, avoid interpolating `${{ inputs.* }}` directly into scripts, and keep code `shellcheck`-clean.
- **Least Privilege:** Request only the GitHub permissions a workflow needs.
- **Idempotency:** Deployment and comment steps must be safe to re-run.
- **Secret Handling:** Never commit credentials, API keys, private tokens, or secrets. Document required secrets in `README.md` and pass them via `secrets: inherit` or organization-level secrets.

---

## Security Vulnerability Disclosures

If you discover a security vulnerability or sensitive exploit:
- **Do not open a public GitHub issue, discussion, or PR.**
- Review our [Security Policy](SECURITY.md) for full reporting procedures and response guidelines.
- Report the vulnerability privately via email to: [`lab@eimlab.org`](mailto:lab@eimlab.org).
- Include full technical details and steps to reproduce. We will coordinate a patched release promptly.

---

## Community & Contact

Have questions, ideas, or feedback?
- **Issue Tracker:** [https://github.com/LABEIM/Shared-CICD/issues](https://github.com/LABEIM/Shared-CICD/issues)
- **Lab Email:** [lab@eimlab.org](mailto:lab@eimlab.org)
- **Lab Portal:** [https://eimlab.org](https://eimlab.org)
- **Organization:** Enterprise Infrastructure Management (EIM) Research Laboratory, Telkom University
