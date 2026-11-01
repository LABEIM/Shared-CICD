# GEMINI.md

> **Single source of truth:** all project rules for AI assistants live in
> [`AGENTS.md`](AGENTS.md). Read it in full before any change — it contains the operational workflow,
> quality gates, definition of done, repository architecture, and development standards.

This repository is the **Shared CI/CD Framework** (`LABEIM/shared-ci-cd`): composite GitHub
Actions and reusable workflows for Cloudflare/Vercel deployment, PR previews, and Lighthouse
audits. Validation is `actionlint` plus YAML syntax checks; there is no application build step.

Gemini-specific notes:

- There are no Gemini-only rules today. Do not re-add project rules here — edit
  `AGENTS.md` instead, so every assistant sees the same instructions.
