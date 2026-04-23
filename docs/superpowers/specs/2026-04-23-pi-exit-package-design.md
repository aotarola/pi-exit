# pi-exit Package Design

## Goal

Turn this repo from a project-local pi extension into a redistributable pi package that anyone can install from git or npm.

## Scope

This package will provide exactly one behavior:

- `/exit` as an alias for pi's built-in `/quit`

Out of scope:

- startup notifications
- extra commands
- custom UI
- non-exit-related features

## Current State

The repo currently contains:

- `.pi/extensions/exit.ts` — local project-only extension
- `README.md` — local usage notes

This layout works only as a project-local extension. It is not the desired final redistribution layout.

## Recommended Approach

Convert the repo to a proper pi package using conventional package directories and `package.json` metadata.

### Why this approach

- supports both `pi install npm:@aotarola/pi-exit` and git-based installs
- matches pi package documentation
- avoids duplicate extension definitions
- keeps the package minimal and predictable

## Package Layout

Final repo structure:

- `extensions/exit.ts` — package extension entrypoint
- `package.json` — npm metadata and pi package manifest
- `README.md` — install and usage documentation

Files to remove from the local-only layout:

- `.pi/extensions/exit.ts`

## Extension Design

The extension will:

- register `/exit`
- describe it as an alias for `/quit`
- call `ctx.shutdown()` in the handler

No additional commands, startup notifications, or side effects will be included.

## Packaging Design

`package.json` will include:

- package name: `@aotarola/pi-exit`
- version: `0.1.0`
- keywords including `pi-package`
- a `pi` manifest pointing to `./extensions`
- peer dependency on `@mariozechner/pi-coding-agent` with `"*"`

The package should be installable as:

- npm package
- git repository package
- temporary extension via `pi -e`

## README Design

The README will document:

- what the package does
- npm install command
- git install command
- one-off testing with `pi -e`
- expected behavior after loading

The README should describe package usage, not the previous project-local `.pi/extensions` flow.

## Git and GitHub Flow

After the package layout is in place:

1. initialize or verify git state
2. commit the packaged version of the repo
3. create a public GitHub repository with `gh`
4. add `origin`
5. push the default branch

## Error Handling

Expected failure cases to handle during setup:

- `gh` not authenticated
- npm package name already taken when publishing later
- git remote creation failure

For this task, GitHub repository creation and push should be attempted. If `gh` auth or repository creation fails, stop and report the exact failure.

## Testing and Verification

Verification should include:

- confirm `extensions/exit.ts` exists
- confirm `package.json` contains a valid `pi` manifest
- confirm local-only `.pi/extensions/exit.ts` is removed
- confirm git status shows intended files
- confirm GitHub remote exists after repo creation
- confirm push succeeds

Runtime behavior can be tested by loading the package with:

- `pi -e .`

and then verifying `/exit` is available.

## Success Criteria

The work is complete when:

- this repo is a valid pi package
- the only packaged feature is `/exit`
- the repo is public on GitHub
- the default branch is pushed
- the repo README explains installation and use for redistribution
