# Phase 6 — Archive

> **Todo:** mark `Phase 6 · Archive` `in_progress` (see `flows/todo-tracking.md`); mark it `completed` at the STOP step.

Only enter when the user gives an explicit "archive" instruction after `APPROVED`.

## 0. The approval must cover what is being archived

`APPROVED` applies to the exact commit the final review looked at. Before anything else:

```
git rev-parse HEAD
```

Compare with `review.md`'s `**Reviewed HEAD:**`.

- **Equal** → continue.
- **Field missing** (review written before this check existed) → announce it and continue.
- **Different** → someone committed, amended or rebased after the review. Show `git log --oneline <reviewed_head>..HEAD` (or say the reviewed commit is no longer an ancestor of HEAD, if `git merge-base --is-ancestor` fails) and ask (AskUserQuestion):
  - **Review the new commits** → rename `review.md` → `review-r<N>.md` and go back to Phase 5; it runs as round N+1 with `FOCUS_RANGE = <reviewed_head>..HEAD`.
  - **Archive anyway** → continue, and add the line `Unreviewed: <short hashes after reviewed_head>` to the commit message in 1b, so the history records that these commits were never reviewed.
  - **Stop** → pause.

Also confirm every item that needs one has a line under `## Dispositions` (Phase 5 section 7). If any is missing, ask for it now.

## 1. Compose the archive commit message

Build the PR-ready summary that the archive commit will carry. Do this before running `openspec archive`.

### 1a. Gather inputs

Step 4 squashes **all** implementation commits into this one commit, so the message must describe **what was actually delivered** — the delivered commits are the PRIMARY source, not the plan.

- Baseline = parent of the `openspec(<name>): planning` commit:
  ```
  BASELINE=$(git rev-parse "$(git log --grep='openspec(<name>): planning' --format='%H' -n 1)~1")
  ```
- **Implementation commits (PRIMARY source — read subject AND body, not just subject):**
  ```
  git log "$BASELINE"..HEAD --format='%h %s%n%b%n---' --no-merges
  ```
  Exclude `openspec(<name>):` scaffolding (planning / final-review commits); keep `feat`/`fix`/`refactor`/`perf`/etc. These commit messages are what you condense the title and Key changes from.
- From `proposal.md`: extract `## Why` and `## Mode` only. `## Why` seeds the `Why:` line; `## Mode` seeds the footer. **Do NOT source `What`/title from `proposal.md`** — the plan may have drifted from what shipped; describe the delivery.
- Count implementation commits (`M`) and tasks (from final `tasks.md`, `N`).
- Confirm `review.md` verdict = `APPROVED`.
- From `review.md`: the `**Verification:**` line (seeds `Testing:`), the `**Review depth:**` line (seeds `Risk:`), and whether `## Accepted open issues` exists.
- From `design.md` `## Verification`: the Full command, if the review line says none ran.

### 1b. Build the message (HARD CAP: 18 lines total, including blanks; 19 when the `Unreviewed:` line is present)

#### Title — derive from the delivered commits, do NOT hardcode

The squashed commit **is the feature delivery** (all the `feat`/`fix`/`refactor` code, not just the openspec file moves), so its title must read like the feature — not like a `chore`. Compose `<type>(<scope>): <subject>` from the implementation commits gathered in 1a:

- **type**: the single Conventional Commit type that best represents the whole delivery. Priority: any new user-facing capability → `feat`; else predominantly bug fixes → `fix`; else restructuring → `refactor`; else the dominant type (`perf`/etc.).
- **scope**: the module/area touched by most of the diff (e.g. `classswift`, `annotation`, `app`). Omit the scope if the work genuinely spans many modules with no dominant one.
- **subject**: a Traditional Chinese phrase (≤ ~60 chars) distilling *what shipped*, condensed from the implementation commit subjects — not copied from one commit, and not from `proposal.md`.

**Fallback (docs/spec-only changes):** if 1a found **no** substantive implementation commits (only `openspec`/`docs` scaffolding, so step 4 will skip the squash), the commit really is just an archive — title it `chore(openspec): archive <name>` and skip the `Key changes` block.

#### Structure

```
<type>(<scope>): <subject distilled from delivered commits>

<one or two sentences condensed from the actual implementation commits — what shipped>

Why: <one sentence (may draw from proposal ## Why)>

Key changes:
- <bullet condensed from the delivered commits, area 1>
- <bullet condensed from the delivered commits, area 2>
- <bullet condensed from the delivered commits, area 3>

Testing: <full verification command> → pass   (or: none defined — reviewed by reading only)
Risk: <Quick|Standard|Deep> review<; touched: sensitive paths matched, if Deep><; N accepted open issues, if any>

Archive: openspec/changes/<name>
Mode: <TDD (Sonnet)|TDD (Opus)|Simple> · Tasks: <N> · Commits: <M> · Review: APPROVED (round <R>)
```

Add `Unreviewed: <hashes>` directly after `Risk:` only when step 0 chose "Archive anyway". Revert needs no line of its own: the whole change is this one commit, so `git revert <hash>` undoes it.

Body rules: condense from the commit subjects+bodies read in 1a (group by area, **not** 1:1 to commits). Bullets: 3 ideal (max 4), ≤ 80 chars each, lead with a verb. Note any intentionally-skipped tasks in one extra line if relevant. If the message exceeds the cap, tighten the What/Why/bullets — never drop the `Testing:`, `Risk:`, `Archive:` or `Mode: ...` lines.

Store the final message as `{COMMIT_MESSAGE}` for the next step.

## 1c. Carry pre-existing issues into the project backlog

Collect, verbatim:
- every bullet under `**Pre-existing ([Pre-existing]):**` in `tasks.md`
- every bullet under `## Pre-existing` in `review.md`
- every `## Dispositions` line answered **Fix later**

If there are none, skip. Otherwise append to `openspec/backlog.md` (create it with a `# Backlog` heading if missing):

```
## <YYYY-MM-DD> — found during <name>
- [Pre-existing] <file:line — issue>
- [Fix later] <item>
```

Do not commit separately — the file is under `openspec/`, so step 3's `git add openspec/` stages it and step 4 squashes it into the archive commit. Each entry is a candidate for its own future `/super-spec` change.

## 2. Run the archive command

```
openspec archive <name>
```

This moves `openspec/changes/<name>/` into `openspec/changes/archive/` and merges the change's spec deltas into `openspec/specs/` (the living spec). The uncommitted `review.md` from Phase 5 gets moved along and is absorbed by step 3's `git add openspec/`.

## 3. Dispatch the archive-committer subagent

Use the **Agent** tool. Read `prompts/archive-committer.md` and substitute:
- `{CHANGE_NAME}`: `<name>`
- `{COMMIT_MESSAGE}`: the multi-line message from step 1b (pass verbatim, preserve newlines)

**Model:** `haiku`

**Forbidden:** any phase conversation; reviewer reports; `proposal.md`. Everything the subagent needs is already condensed into `{COMMIT_MESSAGE}` — it must not re-derive or edit the message.

The subagent stages the archive file moves with `git add openspec/` and commits using `{COMMIT_MESSAGE}` verbatim.

## 4. Squash all commits since baseline

Delegate to a **haiku** Agent subagent. Include the standard CLAUDE.md delegating preamble, then:

```
你是被主 session 委派的子代理，可以直接用 Bash 執行 git 命令，不需要再往下委派。
```

Subagent job:

1. Locate the baseline:
   ```
   PLANNING_HASH=$(git log --oneline --grep="openspec(<name>): planning" --format="%H" -n 1)
   BASELINE=$(git rev-parse "${PLANNING_HASH}~1")
   ```
2. Count commits: `COUNT=$(git rev-list --count "$BASELINE"..HEAD)`
3. If `COUNT` ≤ 1, output `Skipped: only 1 commit since baseline` and stop.
4. Soft-reset to baseline (keeps all changes staged):
   ```
   git reset --soft "$BASELINE"
   ```
5. Re-commit with `{COMMIT_MESSAGE}` verbatim (use a heredoc):
   ```
   git commit -m "$(cat <<'CMEOF'
   {COMMIT_MESSAGE}
   CMEOF
   )"
   ```
6. Output: `Done: <new_commit_hash> (squashed <COUNT> commits)`

On any git failure, report stderr verbatim and halt — no `restore`/`reset --hard`/hook-bypass recovery.

Before dispatching, substitute in the subagent prompt:
- Every `<name>` → the actual change name (same name used in steps 2 and 3).
- `{COMMIT_MESSAGE}` → the exact multi-line message from step 1b.

## 4b. Recurring findings → project rules (asks; never automatic)

Repeated review findings are cheapest to fix at the source. Collect the classes:
- this change: every `**Fixed during review:**` line in the archived `tasks.md`, and every `{class: …}` in the archived `review*.md` files
- history: the same lines across `openspec/changes/archive/*/tasks.md` and `*/review*.md`

Drop `verification-failure` and `unclassified`. A class qualifies when it appears **≥ 2 times in this change** or **≥ 3 times across history**.

If none qualify, skip. Otherwise list them with counts and ask (AskUserQuestion, multiSelect): "These kinds of mistakes keep coming up. Add a check for them to REVIEW.md?" For each selected class, add one line under `## Always check` in `REVIEW.md` (create it from `templates/REVIEW.md` if missing) describing what to look for, and commit separately by delegating to a haiku Agent subagent (same opening line and failure rules as step 4; `git add` only the REVIEW.md path): `docs(review): add recurring checks from <name>`. This commit lands after the archive commit on purpose — it is a project-rule change, not part of the feature. If a class is better caught by a linter rule or a CLAUDE.md instruction, say so in one line instead of adding it.

## 5. STOP

> **Todo:** mark `Phase 6 · Archive` `completed` — all items are now done.

Do not merge. Do not push. Do not open a PR.

Output to user:
```
Archive complete for <name>.
Commit: <hash>
Current branch: <branch>. No merge / push / PR performed (per design).
The archive commit message is PR-ready — paste it into the PR body when you open one.
<if step 1c ran:> Backlog: <k> pre-existing / fix-later items added to openspec/backlog.md.
```
