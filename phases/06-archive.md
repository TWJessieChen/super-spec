# Phase 6 — Archive

Only enter when the user gives an explicit "archive" instruction after `APPROVED`.

1. Run: `openspec archive <name>`

   This moves `openspec/changes/<name>/` into `openspec/changes/archive/` and may update `openspec/specs/`.

2. Dispatch the archive-committer subagent.

   Use the **Agent** tool. Read `prompts/archive-committer.md` and substitute `{CHANGE_NAME}`.

   **Model:** `claude-haiku-4-5-20251001`

   **Forbidden:** any phase conversation; proposal/design/tasks content; reviewer reports.

   The subagent runs `git status` / `git diff --staged`, stages the archive's file moves with `git add openspec/`, and commits with the exact message: `chore(openspec): archive <name>`.

3. **STOP.** Do not merge. Do not push. Do not open a PR.

Output to user:
```
Archive complete for <name>.
Current branch: <branch>. No merge / push / PR performed (per design).
```
