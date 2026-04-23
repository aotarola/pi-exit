# pi-exit Package Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Convert this repo into a redistributable pi package named `@aotarola/pi-exit` that provides `/exit` as an alias for `/quit`, then publish the code to a public GitHub repository.

**Architecture:** Replace the project-local `.pi/extensions` layout with a package layout based on a top-level `extensions/` directory and a `package.json` pi manifest. Keep the extension logic minimal and side-effect free, then document both permanent installation and one-off testing in the README before committing and pushing the repo.

**Tech Stack:** TypeScript, pi extension API, npm package metadata, git, GitHub CLI (`gh`)

---

### Task 1: Convert the extension to package layout

**Files:**
- Create: `extensions/exit.ts`
- Delete: `.pi/extensions/exit.ts`
- Test: `extensions/exit.ts`

- [ ] **Step 1: Write the target extension file content**

```ts
import type { ExtensionAPI } from "@mariozechner/pi-coding-agent";

export default function (pi: ExtensionAPI) {
  pi.registerCommand("exit", {
    description: "Alias for /quit",
    handler: async (_args, ctx) => {
      ctx.shutdown();
    },
  });
}
```

- [ ] **Step 2: Create the package extension file**

Run:
```bash
mkdir -p extensions
cat > extensions/exit.ts <<'EOF'
import type { ExtensionAPI } from "@mariozechner/pi-coding-agent";

export default function (pi: ExtensionAPI) {
  pi.registerCommand("exit", {
    description: "Alias for /quit",
    handler: async (_args, ctx) => {
      ctx.shutdown();
    },
  });
}
EOF
```

Expected: `extensions/exit.ts` exists with the exact command registration above.

- [ ] **Step 3: Remove the local-only extension copy**

Run:
```bash
rm -f .pi/extensions/exit.ts
rmdir .pi/extensions 2>/dev/null || true
rmdir .pi 2>/dev/null || true
```

Expected: `.pi/extensions/exit.ts` no longer exists.

- [ ] **Step 4: Verify the package extension file and removal**

Run:
```bash
test -f extensions/exit.ts && ! test -f .pi/extensions/exit.ts
```

Expected: command exits with code `0`.

- [ ] **Step 5: Commit the layout change**

Run:
```bash
git add extensions/exit.ts .pi/extensions/exit.ts
git commit -m "refactor: move exit extension to package layout"
```

Expected: a commit is created.

### Task 2: Add package metadata for pi and npm

**Files:**
- Create: `package.json`
- Test: `package.json`

- [ ] **Step 1: Write the package manifest content**

```json
{
  "name": "@aotarola/pi-exit",
  "version": "0.1.0",
  "description": "pi package that adds /exit as an alias for /quit",
  "keywords": ["pi-package", "pi", "extension"],
  "license": "MIT",
  "pi": {
    "extensions": ["./extensions"]
  },
  "peerDependencies": {
    "@mariozechner/pi-coding-agent": "*"
  }
}
```

- [ ] **Step 2: Create `package.json`**

Run:
```bash
cat > package.json <<'EOF'
{
  "name": "@aotarola/pi-exit",
  "version": "0.1.0",
  "description": "pi package that adds /exit as an alias for /quit",
  "keywords": ["pi-package", "pi", "extension"],
  "license": "MIT",
  "pi": {
    "extensions": ["./extensions"]
  },
  "peerDependencies": {
    "@mariozechner/pi-coding-agent": "*"
  }
}
EOF
```

Expected: `package.json` exists with the exact metadata above.

- [ ] **Step 3: Verify the manifest loads as JSON and contains the pi extension path**

Run:
```bash
python3 - <<'PY'
import json
with open('package.json') as f:
    data = json.load(f)
assert data['name'] == '@aotarola/pi-exit'
assert data['pi']['extensions'] == ['./extensions']
assert data['peerDependencies']['@mariozechner/pi-coding-agent'] == '*'
print('package.json ok')
PY
```

Expected: prints `package.json ok`.

- [ ] **Step 4: Commit the package manifest**

Run:
```bash
git add package.json
git commit -m "build: add pi package manifest"
```

Expected: a commit is created.

### Task 3: Rewrite README for redistribution

**Files:**
- Modify: `README.md`
- Test: `README.md`

- [ ] **Step 1: Write the README content for package distribution**

```md
# @aotarola/pi-exit

A tiny pi package that adds `/exit` as an alias for `/quit`.

## Install

### From npm

```bash
pi install npm:@aotarola/pi-exit
```

### From git

```bash
pi install git:https://github.com/aotarola/pi-exit
```

## Try without installing

### From npm

```bash
pi -e npm:@aotarola/pi-exit
```

### From git

```bash
pi -e git:https://github.com/aotarola/pi-exit
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
```

- [ ] **Step 2: Replace `README.md` with the distribution README**

Run:
```bash
cat > README.md <<'EOF'
# @aotarola/pi-exit

A tiny pi package that adds `/exit` as an alias for `/quit`.

## Install

### From npm

```bash
pi install npm:@aotarola/pi-exit
```

### From git

```bash
pi install git:https://github.com/aotarola/pi-exit
```

## Try without installing

### From npm

```bash
pi -e npm:@aotarola/pi-exit
```

### From git

```bash
pi -e git:https://github.com/aotarola/pi-exit
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
EOF
```

Expected: `README.md` documents npm install, git install, and `pi -e` usage.

- [ ] **Step 3: Verify the README mentions all required install modes**

Run:
```bash
grep -F "pi install npm:@aotarola/pi-exit" README.md && \
grep -F "pi install git:https://github.com/aotarola/pi-exit" README.md && \
grep -F "pi -e ." README.md
```

Expected: all three `grep` commands print matching lines.

- [ ] **Step 4: Commit the README rewrite**

Run:
```bash
git add README.md
git commit -m "docs: add redistribution instructions"
```

Expected: a commit is created.

### Task 4: Verify the package can be loaded locally

**Files:**
- Test: `package.json`
- Test: `extensions/exit.ts`
- Test: `README.md`

- [ ] **Step 1: Verify the package file set matches the intended layout**

Run:
```bash
find . -maxdepth 2 \( -path './.git' -o -path './docs' \) -prune -o -type f | sort
```

Expected output includes at least:
```text
./README.md
./extensions/exit.ts
./package.json
```

- [ ] **Step 2: Verify the package is loadable as a local pi extension source**

Run:
```bash
pi -e . --help >/tmp/pi-exit-help.txt 2>&1 || cat /tmp/pi-exit-help.txt
```

Expected: command exits successfully or prints standard pi help without package resolution errors.

- [ ] **Step 3: Verify git status contains only intended tracked changes before final commit**

Run:
```bash
git status --short
```

Expected: only package-related files appear.

- [ ] **Step 4: Commit any remaining verification-driven edits**

Run:
```bash
git add README.md extensions/exit.ts package.json docs/superpowers/specs/2026-04-23-pi-exit-package-design.md docs/superpowers/plans/2026-04-23-pi-exit-package.md
git commit -m "test: verify local pi package layout"
```

Expected: a commit is created if there are remaining uncommitted changes.

### Task 5: Create and push the public GitHub repository

**Files:**
- Modify: `.git/config`
- Test: git remote state

- [ ] **Step 1: Verify `gh` authentication is available**

Run:
```bash
gh auth status
```

Expected: authenticated GitHub CLI output for the target account.

- [ ] **Step 2: Create the public GitHub repository and push**

Run:
```bash
gh repo create aotarola/pi-exit --public --source=. --remote=origin --push
```

Expected: GitHub repository is created, `origin` is configured, and the current branch is pushed.

- [ ] **Step 3: Verify the remote URL**

Run:
```bash
git remote -v
```

Expected output includes `origin` pointing at `github.com/aotarola/pi-exit`.

- [ ] **Step 4: Verify the current branch tracks the remote**

Run:
```bash
git status --branch --short
```

Expected: output shows the current branch and tracking information without uncommitted changes.

- [ ] **Step 5: If the current branch is not pushed, push explicitly**

Run:
```bash
git push -u origin HEAD
```

Expected: branch is up to date on GitHub.
