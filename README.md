# Shared CI/CD Framework

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![CI/CD](https://github.com/LABEIM/Shared-CICD/actions/workflows/lint-ci.yml/badge.svg)](https://github.com/LABEIM/Shared-CICD/actions/workflows/lint-ci.yml)
[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-2.1-4baaaa.svg)](CODE_OF_CONDUCT.md)
[![Security Policy](https://img.shields.io/badge/Security-Policy-brightgreen.svg)](SECURITY.md)
[![Organization](https://img.shields.io/badge/Org-LABEIM-6366f1.svg)](https://eimlab.org)

A production-grade, modular CI/CD framework for web applications: composable GitHub Actions and reusable workflows for multi-cloud deployment to Cloudflare Workers and Vercel, pull-request previews, Lighthouse audits, and security scanning.

[**Usage Guide**](USAGE.md) · [**EIM Research Lab**](https://eimlab.org) · [**Report Issue**](https://github.com/LABEIM/Shared-CICD/issues)

---

## About This Project

This repository provides shared CI/CD building blocks consumed by LABEIM projects:

- **Composable actions** under `actions/` for runtime setup and builds, Cloudflare Workers and Vercel deployments, pull-request preview comments, and Lighthouse CI audits.
- **Reusable workflows** under `.github/workflows/` for the full deployment pipeline, CodeQL analysis, dependency review, stale issue/PR cleanup, Dependabot auto-merge, and release tag management.
- **Ready-to-copy examples** under `examples/` covering zero-dependency static sites, framework applications, monorepos, multi-cloud deployments, and remote platform builds.
- **Build-time configuration** — inherited secrets named `PUBLIC_*` are exported into the build environment (e.g. Astro/Vite `import.meta.env.PUBLIC_*`), so consumer config reaches the static build without `with:` inputs.
- **Keystatic CMS on Workers** — an Astro 7 site can deploy its Keystatic Cloud admin at `/keystatic` as a Worker with static assets (`cloudflare_mode: 'ssr'`).
- **Status:** Active
- **Maintained by:** Enterprise Infrastructure Management (EIM) Research Laboratory, Telkom University

---

## Getting Started

Ready to deploy? The [Usage & Deployment Guide](USAGE.md) covers requirements, a minimal caller workflow, deployment patterns, the full input/secret reference, composite actions, framework recipes, and troubleshooting.

---

## Documentation

| Document | Contents |
| :--- | :--- |
| [Usage & Deployment Guide](USAGE.md) | Setup, deployment patterns, inputs, secrets, composite actions, troubleshooting |
| [Release Guide](RELEASING.md) | SemVer strategy and floating major tags |
| [Contributing Guide](CONTRIBUTING.md) | Branching, commit conventions, and the PR workflow |
| [Code of Conduct](CODE_OF_CONDUCT.md) | Community standards and enforcement |
| [Security Policy](SECURITY.md) | Supported versions and vulnerability reporting |

---

## Community & Contributing

We are committed to fostering an open, welcoming, and harassment-free environment for all researchers, developers, and contributors. All participants and team members are expected to adhere to our [Code of Conduct](CODE_OF_CONDUCT.md).

For contribution workflows, git branching standards, and Pull Request guidelines, please refer to our [Contributing Guide](CONTRIBUTING.md).

To report security vulnerabilities responsibly, please review our [Security Policy](SECURITY.md).

For concerns or incident reports, please contact the project team or laboratory administration at [`lab@eimlab.org`](mailto:lab@eimlab.org).

---

## License & Ownership

This project is open-source and licensed under the [MIT License](LICENSE).  
Copyright (c) 2026 **Enterprise Infrastructure Management (EIM) Research Laboratory**, Telkom University.
