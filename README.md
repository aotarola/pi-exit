# @aotarola/pi-exit

A tiny pi package that adds `/exit` as an alias for `/quit`.

## Install

### From npm

```bash
pi install npm:@aotarola/pi-exit
```

### From git

```bash
pi install git:github.com/aotarola/pi-exit
```

## Try without installing

### From npm

```bash
pi -e npm:@aotarola/pi-exit
```

### From git

```bash
pi -e git:github.com/aotarola/pi-exit
```

### From a local checkout

```bash
pi -e .
```

## Usage

After the package is loaded, use either command:

```text
/quit
/exit
```

## What it does

This package registers a single pi slash command:

- `/exit` — graceful alias for `/quit` via `ctx.shutdown()`

## Release

Publishing is handled by GitHub Actions when a version tag is pushed.

One-time setup:

1. In npm, open the package settings for `@aotarola/pi-exit`.
2. Configure a trusted publisher for GitHub Actions:
   - Organization/user: `aotarola`
   - Repository: `pi-exit`
   - Workflow filename: `release.yml`
   - Environment: leave blank unless the workflow is later changed to use a GitHub environment

No npm token is required; GitHub Actions authenticates with npm using OIDC trusted publishing.

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
