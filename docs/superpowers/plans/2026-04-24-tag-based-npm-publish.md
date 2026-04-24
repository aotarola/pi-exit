# Tag-Based npm Publish Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Configure GitHub Actions so the pi extension package validates on normal changes and publishes to npm only from matching `vX.Y.Z` tags.

**Architecture:** Add package publish metadata to `package.json` and a single GitHub Actions workflow at `.github/workflows/release.yml`. The workflow has a validation job for pushes and pull requests, and a publish job gated to semver tags that confirms the tag matches the package version before publishing.

**Tech Stack:** npm package metadata, GitHub Actions, Node.js 24, npm publish with `NPM_TOKEN`.

---

## File Structure

- Modify `package.json`: add npm metadata needed for public publishing and limit packed files.
- Create `.github/workflows/release.yml`: validate package contents and publish on matching tags.
- Modify `README.md`: document the tag-based release flow and required `NPM_TOKEN` secret.

---

### Task 1: Add npm publish metadata

**Files:**
- Modify: `package.json`

- [ ] **Step 1: Inspect current npm package shape**

Run:

```bash
npm pack --dry-run
```

Expected: command succeeds and shows the current tarball contents.

- [ ] **Step 2: Update package metadata**

Replace `package.json` with:

```json
{
  "name": "@aotarola/pi-exit",
  "version": "0.1.0",
  "description": "pi package that adds /exit as an alias for /quit",
  "keywords": ["pi-package", "pi", "extension"],
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/aotarola/pi-exit.git"
  },
  "bugs": {
    "url": "https://github.com/aotarola/pi-exit/issues"
  },
  "homepage": "https://github.com/aotarola/pi-exit#readme",
  "files": [
    "extensions",
    "README.md"
  ],
  "publishConfig": {
    "access": "public"
  },
  "pi": {
    "extensions": ["./extensions"]
  },
  "peerDependencies": {
    "@mariozechner/pi-coding-agent": "*"
  }
}
```

- [ ] **Step 3: Verify package metadata parses**

Run:

```bash
node -e "const p=require('./package.json'); if (p.name !== '@aotarola/pi-exit') throw new Error('bad name'); if (p.publishConfig.access !== 'public') throw new Error('bad access'); if (!p.files.includes('extensions')) throw new Error('missing extensions files entry'); console.log('package metadata ok')"
```

Expected: prints `package metadata ok`.

- [ ] **Step 4: Verify dry-run tarball contents**

Run:

```bash
npm pack --dry-run
```

Expected: output includes `extensions/exit.ts`, `README.md`, and `package.json`; output does not include `.github/` or `docs/`.

- [ ] **Step 5: Commit metadata change**

```bash
git add package.json
git commit -m "build: add npm publish metadata"
```

---

### Task 2: Add GitHub Actions release workflow

**Files:**
- Create: `.github/workflows/release.yml`

- [ ] **Step 1: Create the workflow file**

Create `.github/workflows/release.yml` with:

```yaml
name: Release

on:
  push:
    branches:
      - main
    tags:
      - "v*.*.*"
  pull_request:
    branches:
      - main

permissions:
  contents: read

jobs:
  validate:
    name: Validate package
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 24

      - name: Validate package metadata
        run: |
          node -e "const p=require('./package.json'); if (!p.name) throw new Error('missing package name'); if (!p.version) throw new Error('missing package version'); if (p.publishConfig?.access !== 'public') throw new Error('publishConfig.access must be public'); if (!p.pi?.extensions?.length) throw new Error('missing pi.extensions manifest'); console.log(`${p.name}@${p.version}`)"

      - name: Verify package can be packed
        run: npm pack --dry-run

  publish:
    name: Publish to npm
    needs: validate
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/v')
    permissions:
      contents: read
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 24
          registry-url: https://registry.npmjs.org

      - name: Verify tag matches package version
        run: |
          PACKAGE_VERSION=$(node -p "require('./package.json').version")
          TAG_VERSION="${GITHUB_REF_NAME#v}"
          if [ "$PACKAGE_VERSION" != "$TAG_VERSION" ]; then
            echo "Tag v$TAG_VERSION does not match package.json version $PACKAGE_VERSION" >&2
            exit 1
          fi
          echo "Publishing version $PACKAGE_VERSION"

      - name: Publish package
        run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

- [ ] **Step 2: Verify workflow YAML exists and contains the release gate**

Run:

```bash
test -f .github/workflows/release.yml && grep -q 'refs/tags/v' .github/workflows/release.yml && grep -q 'secrets.NPM_TOKEN' .github/workflows/release.yml && echo 'workflow release gate ok'
```

Expected: prints `workflow release gate ok`.

- [ ] **Step 3: Verify package validation command locally**

Run:

```bash
node -e "const p=require('./package.json'); if (!p.name) throw new Error('missing package name'); if (!p.version) throw new Error('missing package version'); if (p.publishConfig?.access !== 'public') throw new Error('publishConfig.access must be public'); if (!p.pi?.extensions?.length) throw new Error('missing pi.extensions manifest'); console.log(`${p.name}@${p.version}`)"
```

Expected: prints `@aotarola/pi-exit@0.1.0`.

- [ ] **Step 4: Commit workflow**

```bash
git add .github/workflows/release.yml
git commit -m "ci: publish package on version tags"
```

---

### Task 3: Document release flow

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Add release documentation**

Append this section to `README.md`:

```markdown
## Release

Publishing is handled by GitHub Actions when a version tag is pushed.

One-time setup:

1. Create an npm automation token or publish token for `@aotarola/pi-exit`.
2. Add it to this GitHub repository as the secret `NPM_TOKEN`.

Release a patch version:

```bash
npm version patch
git push --follow-tags
```

Release a minor or major version:

```bash
npm version minor
git push --follow-tags
```

```bash
npm version major
git push --follow-tags
```

The pushed tag must match `package.json`. For example, tag `v0.1.1` publishes package version `0.1.1`.
```

- [ ] **Step 2: Verify README mentions required setup**

Run:

```bash
grep -q 'NPM_TOKEN' README.md && grep -q 'npm version patch' README.md && grep -q 'git push --follow-tags' README.md && echo 'release docs ok'
```

Expected: prints `release docs ok`.

- [ ] **Step 3: Commit docs**

```bash
git add README.md
git commit -m "docs: document release process"
```

---

### Task 4: Final verification

**Files:**
- Verify: `package.json`
- Verify: `.github/workflows/release.yml`
- Verify: `README.md`

- [ ] **Step 1: Run package dry-run**

Run:

```bash
npm pack --dry-run
```

Expected: command succeeds and tarball contains only publishable package files.

- [ ] **Step 2: Run metadata validation**

Run:

```bash
node -e "const p=require('./package.json'); if (p.name !== '@aotarola/pi-exit') throw new Error('bad name'); if (!/^\\d+\\.\\d+\\.\\d+$/.test(p.version)) throw new Error('bad version'); if (p.publishConfig.access !== 'public') throw new Error('bad publish access'); if (!p.pi.extensions.includes('./extensions')) throw new Error('bad pi extensions manifest'); console.log('final metadata ok')"
```

Expected: prints `final metadata ok`.

- [ ] **Step 3: Confirm working tree state**

Run:

```bash
git status --short
```

Expected: no uncommitted changes except ignored local files, if any.

- [ ] **Step 4: Report release command**

Tell the user:

```bash
npm version patch
git push --follow-tags
```

Also remind them to add `NPM_TOKEN` in GitHub repository secrets before pushing a release tag.

---

## Self-Review

- Spec coverage: The plan covers npm metadata, validation on PR/push, tag-gated publish, tag/package version matching, `NPM_TOKEN`, dry-run validation, and release docs.
- Placeholder scan: No TBD/TODO/fill-in placeholders remain.
- Type and command consistency: File paths, commands, package names, and version checks are consistent across tasks.
