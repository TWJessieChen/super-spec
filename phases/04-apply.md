# Phase 4 — Apply

> **Todo:** at Phase 4 entry, rewrite the list from Shape A to **Shape B** — expand `Phase 4 · Apply tasks` into one `Phase 4 · Task N: <title>` item per Task in `tasks.md`, with already-`- [x]` Tasks marked `completed` (see `flows/todo-tracking.md`). Do this once, after the sweep below and before the Task loop.

## 0. Phase-entry sweep (once, at entry only)

**Only at Phase 4 entry** (fresh or resumed — not on per-Task re-Reads of this file): Read `flows/phase4-sweep.md` and execute it — it purges superpowers artifacts and may rewrite history, so it runs orchestrator-side in BOTH engines, before anything else in this phase. If this Read happens at the top of a Task iteration (the sweep already ran at entry), skip straight to the Task loop below.

---

## Engine routing (after the sweep, before the Task loop)

Read `## Engine` from `openspec/changes/<name>/proposal.md`:

- **`ultracode`** → Read `flows/ultracode-apply.md` and execute it **instead of** the Task loop below (steps 1–5 are the native path; do not run them).
- **`native` or the section is absent** (changes created before engines existed) → continue with the Task loop below.

## Task loop

Loop over each Task in `tasks.md` that is not yet complete (`- [ ]` at the Task header level).

For each Task, do steps 1 → 4 in order. Keep these per-Task counters in the conversation (they reset for every Task):

| Counter | Meaning |
|---|---|
| `failCount` | consecutive FAILs (reviewer FAIL **or** verification failure) — drives escalation and the cap |
| `reviewRound` | how many times the task-reviewer has been dispatched for this Task |
| `verifyFailures` | how many times the per-task verification command failed |
| `lastReviewedCommit` | the commit the most recent reviewer round looked at |
| `lastGatingIssues` | that round's `[Critical]` / `[Important]` lines, verbatim |
| `fixedClasses` | every `{class: …}` seen in a FAIL round for this Task |

## 1. Dispatch the implementer subagent

> **Todo:** mark this Task's `Phase 4 · Task N` item `in_progress` before dispatching. Re-dispatches and FAIL rounds keep it `in_progress`.

Use the **Agent** tool (not Skill — we need fresh-context isolation).

Read `prompts/implementer.md` (relative to this skill's directory) and substitute the variables. Pass the resulting text as the agent's prompt.

Variables:
- `{TASK_NUMBER}`: Task index (e.g., 3)
- `{TASK_BODY}`: This Task's section from tasks.md only (header + sub-steps)
- `{DESIGN_PATH}`: `openspec/changes/<name>/design.md`
- `{MODE}`: `TDD` or `Simple` — the **discipline** only. Strip any model qualifier from `proposal.md`'s `## Mode` line: both `TDD (Sonnet)` and `TDD (Opus)` map to `TDD` here.
- `{RELEVANT_FILES}`: list of existing files this Task touches (extract from the Task body's `**Files:**` section)

**Model:** read the parenthetical on `proposal.md`'s `## Mode` line (use Agent-tool aliases, not full model IDs — aliases track the latest version automatically):
- `TDD (Sonnet)` → `sonnet`
- `TDD (Opus)` → `opus`
- `Simple` → inherit (omit the `model` parameter)

**Escalation override:** if this dispatch is a re-dispatch after the **2nd consecutive FAIL** on this Task, use `opus` regardless of the Mode line (see "Model escalation on consecutive FAIL" below).

**Re-dispatch payload:** after a FAIL, append a `## Previous review failed with these issues` section containing **only the blocking lines** — the `[Critical]` / `[Important]` issues (with their `{class: …}` tags) or the failed-verification issue from step 1b — plus a `Previous commit: <commit_hash>` line. **Never include `[Minor]` or `[Pre-existing]` items**: an agent author treats every line as work, and each extra edit creates more for the next round to review.

**Forbidden in this prompt** (do not include):
- Brainstorming conversation
- `proposal.md` content (apart from the Mode flag, which is passed via `{MODE}`)
- Other Tasks' content
- Other Tasks' reviewer reports
- Any other implementer's response or narrative

The implementer is contractually required to return one line: `Done: <commit_hash>` or `Blocked: <reason>`.

### Blocked routing

If the implementer returns `Blocked: <reason>`, do NOT dispatch a reviewer. Pause the workflow: surface the reason verbatim and ask the user how to proceed (e.g., adjust the design, rewrite this Task's body, skip it, intervene manually). A `Blocked` return does not count toward the consecutive-FAIL counter.

## 1b. Run the per-task verification command (machine gate, before any reviewer)

Read `design.md`'s `## Verification` section and take the **Per-task** command.

- **No section, or `(none)`** → skip this step; `{VERIFICATION}` = `(no per-task verification command defined in design.md)`.
- **Otherwise** → delegate to a **haiku** Agent subagent. Prompt must contain:
  - Opening: `你是被主 session 委派的子代理，可以直接用 Bash 執行命令，不需要再往下委派。`
  - Job: run the command verbatim from the repo root (generous timeout — builds can take minutes); do not edit any file; report exactly two things: `exit_code: <n>` and the last 40 lines of combined stdout/stderr inside a fenced block.
  - On being unable to run it at all (command not found, etc.): report `exit_code: -1` plus the error.

Route on the exit code:

- **0** → `{VERIFICATION}` = `` `<command>` → exit 0 `` followed by the fenced output tail. Continue to step 2.
- **non-zero** → do **not** dispatch a reviewer. This counts as a FAIL: `failCount += 1`, `verifyFailures += 1`. Build the blocking issue
  `- [Critical] verification command failed: \`<command>\` exited <n> — output tail below {class: verification-failure}` followed by the fenced tail, and go to "Severity-based routing → FAIL" below with that single issue as the reviewer output. `lastReviewedCommit` / `lastGatingIssues` stay unchanged (no reviewer looked at this commit).

## 2. Dispatch the task-reviewer subagent

Use the **Agent** tool. Read `prompts/task-reviewer.md` and substitute:
- `{COMMIT_HASH}`: the hash from the implementer's response
- `{DESIGN_PATH}`: `openspec/changes/<name>/design.md`
- `{TASK_BODY}`: the same Task section
- `{MODE}`: the same discipline flag passed to the implementer (`TDD` or `Simple`)
- `{REVIEW_ROUND}`: `reviewRound + 1`
- `{PREVIOUS_COMMIT}`: `lastReviewedCommit`, or `(none)` in round 1
- `{PREVIOUS_ISSUES}`: `lastGatingIssues` verbatim, or `(none)` in round 1
- `{VERIFICATION}`: from step 1b

Then set `reviewRound += 1`, `lastReviewedCommit = <this commit>`.

**Model:** `sonnet`

**Forbidden:** the implementer's narrative; other Tasks' content; `[Minor]` / `[Pre-existing]` items from earlier rounds.

The reviewer returns one of:

- `PASS` — no issues at all.
- `PASS` followed by a bullet list of `[Minor]` and/or `[Pre-existing]` items — Task passes; record them (see "Recording" below) but do NOT re-dispatch.
- `FAIL` followed by a bullet list — at least one `[Critical]` or `[Important]` issue exists; re-dispatch (see below). Ordered `[Critical]` → `[Important]` → `[Minor]` → `[Pre-existing]`; each blocking line ends with `{class: …}`.

### Severity-based routing

- **No `[Critical]` and no `[Important]`** → Task passes. Record per "Recording" below, then proceed to step 3 (mark complete) and step 4 (fold into the implementer commit).
- **Any `[Critical]` or `[Important]`** (or a verification failure from step 1b) → `failCount += 1` (if not already counted in 1b); set `lastGatingIssues` to the blocking lines (reviewer FAIL only); add each `{class: …}` to `fixedClasses`; then re-dispatch the implementer (step 1) with the **blocking lines only** under `## Previous review failed with these issues`, plus a `Previous commit: <commit_hash>` line. The re-dispatched implementer fixes by **amending** that commit (it sits at HEAD), so each Task still ends with a single implementation commit; the amended hash becomes the new `{COMMIT_HASH}` for re-verification and re-review.

The implementer is required to address every `[Critical]` and `[Important]` issue, and every instance of each issue's class. `[Minor]` and `[Pre-existing]` items are never sent back.

### Malformed reviewer output

If the reviewer's output does not start with `PASS` or `FAIL`, or contradicts the severity rules (e.g., `FAIL` with only `[Minor]` / `[Pre-existing]` items, `PASS` listing a `[Critical]`/`[Important]`, or a blocking line with no `{class: …}`), do not silently reinterpret it: re-dispatch the task-reviewer once (same round number) with a note pointing out the inconsistency. If the second output is still malformed, pause and surface both outputs to the user. Malformed rounds do not count toward the consecutive-FAIL counter.

### Model escalation on consecutive FAIL

Track consecutive FAILs per Task. The implementer model per attempt:

- **1st FAIL → 2nd dispatch:** same model as the Mode line (a re-try with the reviewer report is usually enough).
- **2nd FAIL → 3rd dispatch:** escalate the implementer to `opus` (if not already `opus`). A model stuck twice on the same blind spot rarely fixes it on the third identical attempt — one `opus` dispatch is cheaper than burning another failed round plus the human-intervention pause.
- The task-reviewer stays on `sonnet` throughout — only the implementer escalates.
- The escalation is per-Task and resets on the next Task. It does NOT modify `proposal.md`'s `## Mode` line.

### Recording (on PASS)

Append to the Task's section in `tasks.md`, in this order (omit a block when it would be empty):

```
**Review rounds:** <reviewRound> · **Verification failures:** <verifyFailures>
**Fixed during review:** <comma-separated fixedClasses>
**Reviewer advisories ([Minor]):**
- <verbatim [Minor] item>
**Pre-existing ([Pre-existing]):**
- <verbatim [Pre-existing] item>
```

The `Review rounds` line is always written (it feeds `## Review stats` in Phase 5). Minor advisories carry forward so the final-reviewer can decide whether they accumulate into a real problem; Pre-existing items are collected into `openspec/backlog.md` at archive time (Phase 6); `Fixed during review` classes feed the recurring-finding check in Phase 6.

### Failure cap

After the **3rd consecutive FAIL** on the same Task, **pause** the workflow. Three rounds on one Task usually means the approach, not the code, is wrong:
- Output the latest reviewer report (or verification output) to the user
- Ask (AskUserQuestion): "This Task failed 3 rounds in a row. How would you like to proceed?" with options:
  - **Revisit the design** — treat as `NEEDS DESIGN UPDATE` for this Task: Read `flows/recover.md` (the Task's blocking issues become the reviewer issues of the revision block)
  - **Rewrite this Task's body** — you edit `tasks.md`, then the Task restarts with fresh counters
  - **Skip this Task** — mark it `- [x]` with `（skipped by user）`
  - **Intervene manually** — you fix it, then say "continue"
- Wait for user direction

## 3. Mark Task complete

> **Todo:** after flipping the checkboxes below, mark this Task's `Phase 4 · Task N` item `completed`.

Use the Edit tool to flip `- [ ]` → `- [x]` for:
- Each sub-step under this Task (if not already done by the implementer)
- The Task header itself

## 4. Fold tasks.md into the implementer commit

If `tasks.md` has uncommitted changes after steps 2–3, **amend them into the implementer's feat commit**. Do NOT create a separate `openspec(<name>): record ...` commit. Skip this step if `git status --porcelain openspec/changes/<name>/tasks.md` is empty.

Delegate to a haiku Agent subagent. Prompt must contain:
- Opening: `你是被主 session 委派的子代理，可以直接用 Bash 執行 git 命令，不需要再往下委派。`
- Commands: `git add openspec/changes/<name>/tasks.md && git commit --amend --no-edit`
- On failure: report stderr verbatim and halt; no `restore` / `reset --hard` / hook-bypass recovery.

Safe to amend: the task-reviewer's verdict is already in; no downstream step references the pre-amend hash.

## 5. Loop

If more Tasks remain with `- [ ]`, return to step 1 with the next Task. Otherwise → Phase 5.
