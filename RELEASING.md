# Release Guide

This document outlines the versioning strategy, release workflow, and tag management procedures for the shared CI/CD repository.

---

## 1. Versioning Strategy

This repository follows [Semantic Versioning 2.0.0](https://semver.org/) (`vMAJOR.MINOR.PATCH`) alongside **floating major tags** (e.g. `v1`, `v2`), which is the standard practice for GitHub Actions and reusable workflows.

| Version Tag Type | Example | When to Use | Downstream Impact |
| :--- | :--- | :--- | :--- |
| **Major (`vX.0.0` / `vX`)** | `v2.0.0`, `v2` | Breaking changes (e.g. removing inputs, changing mandatory secrets, restructuring outputs, or changing the deployment platform) | Requires manual updates in consumer workflows (`@v2`) |
| **Minor (`vX.Y.0`)** | `v2.1.0` | New backwards-compatible features (e.g. new optional inputs, new composite actions) | Automatically inherited by consumers pinned to `@v2` |
| **Patch (`vX.Y.Z`)** | `v2.0.1` | Bug fixes, dependency updates, retry improvements | Automatically inherited by consumers pinned to `@v2` |
| **Floating Tag (`vX`)** | `v2` | Pointers to the latest non-breaking release in the major series | Always references the latest stable `v2.x.x` |

---

## 2. Release Procedures

### Scenario A: Initial Release (`v1.0.0` and `v1`)

When publishing the initial version of the framework:

1. Ensure the `main` branch is clean, tested, and pushed:
   ```bash
   git checkout main
   git pull origin main
   ```

2. Create the exact semantic tag and the floating major tag:
   ```bash
   git tag -a v1.0.0 -m "Initial release: modular hybrid CI/CD framework"
   git tag -fa v1 -m "Release v1"
   ```

3. Push both tags to GitHub:
   ```bash
   git push origin v1.0.0
   git push origin v1 --force
   ```

4. Create a GitHub Release:
   * Go to **Releases** -> **Draft a new release**.
   * Choose tag: `v1.0.0`.
   * Title: `Release v1.0.0`.
   * Click **Generate release notes** and then **Publish release**.

---

### Scenario B: Releasing a Patch or Minor Update (Non-Breaking)

When fixing bugs, updating action dependencies, or adding optional features:

1. Merge your changes to `main`:
   ```bash
   git checkout main
   git pull origin main
   ```

2. Tag the new patch or minor version (e.g. `v2.1.0`):
   ```bash
   git tag -a v2.1.0 -m "feat: add support for Bun and update action dependencies"
   ```

3. Move the floating `v2` tag to point to the new commit:
   ```bash
   git tag -fa v2 -m "Update v2 to v2.1.0"
   ```

4. Push the new semantic tag and force-push the floating `v2` tag:
   ```bash
   git push origin v2.1.0
   git push origin v2 --force
   ```

5. Publish the release notes on GitHub under tag `v2.1.0`.

> [!IMPORTANT]
> `deploy-app.yml` calls this repository's composite actions at a literal `@v2`. Tag the workflow and action changes in the same commit and move `v2` once; a workflow tested from a branch or exact tag before `v2` moves still resolves the previous actions. To test a release candidate end-to-end, temporarily repoint the workflow's `LABEIM/shared-ci-cd/actions/*@v2` refs to your branch (or a prerelease tag), run a canary consumer at that ref, then restore `@v2` before merging.

> **Result**: All downstream repositories referencing `@v2` will automatically inherit the update on their next workflow run without modifying any code.

### v2.3.0 behavior changes

Record these in the release notes, because consumers pinned to `@v2` inherit them without editing anything:

- Production requires a non-`pull_request` run of a branch whose ref equals `production_branch` (default `main`), on both Cloudflare and Vercel. Previously Cloudflare also treated `main`/`master` as production regardless of the input, and Vercel treated every non-PR event as production.
- Vercel no longer deploys production from arbitrary branches, tags, or release events; those deploy previews.
- The GitHub Environment (`*-production` / `*-preview`) follows the same rule, so a tag, release, or `workflow_dispatch`-on-a-tag run now uses the `*-preview` environment. Consumers that scope deployment tokens to the `*-production` environment only must add them to `*-preview` as well.
- The build job runs with `contents: read` and `persist-credentials: false`. A `build_command` that pushes to the repository, or calls `gh`, now fails.
- A `PUBLIC_*` name defined as both a secret and a non-empty variable with a different value now fails the build instead of silently preferring the variable.

---

### Scenario C: Releasing a Breaking Change (`v2.0.0`)

When making breaking changes (e.g. altering required parameters or dropping supported platforms):

1. Merge changes to `main`:
   ```bash
   git checkout main
   git pull origin main
   ```

2. Create the new major semantic tag and the new `v2` floating tag:
   ```bash
   git tag -a v2.0.0 -m "feat!: deploy to Cloudflare Workers static assets (Pages removed)"
   git tag -fa v2 -m "Release v2"
   ```

3. Push the tags:
   ```bash
   git push origin v2.0.0
   git push origin v2 --force
   ```

4. **Do NOT update the `v1` tag**. The `v1` tag must remain pointing to the last stable `v1.x.x` release so that existing repositories do not break.

5. Downstream repositories can migrate when ready by updating their workflow file:
   ```yaml
   uses: your-org/ci-cd-eim/.github/workflows/deploy-app.yml@v2
   ```

---

## 3. Automated Tag Management

This repository ships `.github/workflows/release-tagger.yml`. It runs on `release: released` (not `published`, so a prerelease cannot move the floating tag) and skips the move when the published version is not greater than the highest existing tag in that major series. Publishing an older release therefore cannot drag `@v2` backwards.

The workflow force-pushes only the floating major tag, never the semantic tag.

---

## 4. Release Checklist

Before tagging and publishing any release, verify:

- [ ] All action definitions in `actions/*/action.yml` have valid YAML syntax.
- [ ] New inputs/outputs are properly documented in `README.md`.
- [ ] Example workflows in `examples/` are up to date.
- [ ] CI validation checks on `main` are green.
- [ ] Floating major tag for the release (`v2`) has been pushed.
- [ ] The `wrangler` and `vercel` pins are current (see below).

### Version pins

`wrangler` and `vercel` are pinned inside composite-action shell scripts, so neither Dependabot nor the action review will ever bump them. Before tagging:

```bash
grep -rn 'wrangler@\|vercel@' actions/*/action.yml
npm view wrangler version && npm view vercel version
```

Bump deliberately, in a separate commit, and re-run the `deploy-cloudflare` and `deploy-vercel` checks. A `vercel` major can change the Build Output API behaviour; a `wrangler` major can change the Preview config shape.
