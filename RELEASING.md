# Release Guide

This document outlines the versioning strategy, release workflow, and tag management procedures for the shared CI/CD repository.

---

## 1. Versioning Strategy

This repository follows [Semantic Versioning 2.0.0](https://semver.org/) (`vMAJOR.MINOR.PATCH`) alongside **floating major tags** (e.g. `v1`, `v2`), which is the standard practice for GitHub Actions and reusable workflows.

| Version Tag Type | Example | When to Use | Downstream Impact |
| :--- | :--- | :--- | :--- |
| **Major (`vX.0.0` / `vX`)** | `v2.0.0`, `v2` | Breaking changes (e.g. removing inputs, changing mandatory secrets, restructuring outputs) | Requires manual updates in consumer workflows (`@v2`) |
| **Minor (`vX.Y.0`)** | `v1.1.0` | New backwards-compatible features (e.g. new optional inputs, new composite actions) | Automatically inherited by consumers pinned to `@v1` |
| **Patch (`vX.Y.Z`)** | `v1.0.1` | Bug fixes, dependency updates, retry improvements | Automatically inherited by consumers pinned to `@v1` |
| **Floating Tag (`vX`)** | `v1` | Pointers to the latest non-breaking release in the major series | Always references the latest stable `v1.x.x` |

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

2. Tag the new patch or minor version (e.g. `v1.1.0`):
   ```bash
   git tag -a v1.1.0 -m "feat: add support for Bun and update action dependencies"
   ```

3. Move the floating `v1` tag to point to the new commit:
   ```bash
   git tag -fa v1 -m "Update v1 to v1.1.0"
   ```

4. Push the new semantic tag and force-push the floating `v1` tag:
   ```bash
   git push origin v1.1.0
   git push origin v1 --force
   ```

5. Publish the release notes on GitHub under tag `v1.1.0`.

> **Result**: All downstream repositories referencing `@v1` will automatically inherit the update on their next workflow run without modifying any code.

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
   git tag -a v2.0.0 -m "feat!: major overhaul of pipeline outputs and inputs"
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

## 3. Automated Tag Management (Optional)

To automate updating the floating major tag (`v1`) whenever you publish a release via the GitHub Web UI, you can add `.github/workflows/release-tagger.yml`:

```yaml
name: Release Tag Manager

on:
  release:
    types: [published]

permissions:
  contents: write

jobs:
  update-major-tag:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Update floating major tag
        run: |
          TAG="${{ github.event.release.tag_name }}"
          MAJOR_VERSION=$(echo "$TAG" | grep -oP '^v[0-9]+')
          if [ -n "$MAJOR_VERSION" ]; then
            echo "Updating floating tag $MAJOR_VERSION to point to $TAG..."
            git config user.name "github-actions[bot]"
            git config user.email "github-actions[bot]@users.noreply.github.com"
            git tag -fa "$MAJOR_VERSION" -m "Update $MAJOR_VERSION to $TAG"
            git push origin "$MAJOR_VERSION" --force
          fi
```

---

## 4. Release Checklist

Before tagging and publishing any release, verify:

- [ ] All action definitions in `actions/*/action.yml` have valid YAML syntax.
- [ ] New inputs/outputs are properly documented in `README.md`.
- [ ] Example workflows in `examples/` are up to date.
- [ ] CI validation checks on `main` are green.
- [ ] Floating major tag (`v1`) has been pushed.
