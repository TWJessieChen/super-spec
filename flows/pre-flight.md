# Pre-flight

Run on every `/super-spec` invocation. Halt on first failure.

## 1. openspec CLI present

Run: `openspec --version`

If non-zero exit, halt with:
```
openspec CLI not installed. Install:
  npm i -g @openspec/cli
See https://openspec.dev for details.
```

## 2. openspec/ initialized in current project

Run: `openspec status` (no args)

If it errors because there's no `openspec/` directory, ask the user (use AskUserQuestion):
> "openspec is not initialized in this project. Initialize now?"
Options: `yes` / `no`.
- `yes` → run `openspec init`, continue
- `no` → halt

## 3. No tracked-file changes (untracked files are allowed)

Run: `git status --porcelain --untracked-files=no`

If non-empty, halt with:
```
Tracked files have uncommitted changes. Please commit, stash, or discard them before /super-spec.
(Untracked files are OK and do not block.)

<output of `git status --short --untracked-files=no`>
```

## Branch policy

Do **NOT** check the current branch. Do **NOT** create a new branch. All commits go on the current branch.
