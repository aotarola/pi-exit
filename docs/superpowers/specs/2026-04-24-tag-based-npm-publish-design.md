# Tag-Based npm Publish Design

## Goal

Configure this pi extension package so GitHub Actions validates normal changes and publishes the npm package only when a version tag is pushed.

## Release model

Publishing is tag-based. A maintainer updates `package.json` with `npm version patch`, `npm version minor`, or `npm version major`, then pushes the resulting commit and `vX.Y.Z` tag with `git push --follow-tags`. GitHub Actions treats tags matching `v*.*.*` as release triggers.

## Approach

Use one GitHub Actions workflow with two responsibilities:

1. Validate pushes and pull requests with a lightweight npm package check.
2. Publish to npm only for version tags.

The publish job must verify that the pushed tag matches the `package.json` version before running `npm publish`. For example, tag `v0.1.1` must match package version `0.1.1`.

## Package metadata

The package should include publish-friendly npm metadata:

- `repository`
- `bugs`
- `homepage`
- `files`
- `publishConfig.access: public`

The package should keep its existing `pi` manifest and peer dependency on `@mariozechner/pi-coding-agent`.

## Authentication

The workflow should support npm publishing from GitHub Actions using `NODE_AUTH_TOKEN` sourced from the GitHub secret `NPM_TOKEN`. The repository owner must add an npm automation token or publish token as `NPM_TOKEN` in GitHub repository secrets.

## Testing and validation

Validation should run `npm pack --dry-run` to confirm the package contents and publish shape. Because this package has no build step and no runtime dependencies, the workflow should avoid unnecessary complexity.

## Release flow

```bash
npm version patch
git push --follow-tags
```

The same flow works for `minor` and `major` releases.
