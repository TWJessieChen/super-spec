# Phase 5 — Final Review

> **Todo:** mark `Phase 5 · Final review` `in_progress` (see `flows/todo-tracking.md`).

Phase 5 runs in rounds. Round 1 reviews the whole change; each later round (after revision tasks) only confirms the previous issues are closed and reviews the revision commits. After round 3 the workflow stops adding revision tasks on its own and asks you.

## 1. Round bookkeeping

Compute and keep in the conversation:

- **Round `N`** = 1 + the number of `openspec/changes/<name>/review-r*.md` files.
- **`COMMIT_RANGE`** = `<baseline_hash>..HEAD`, where baseline = the commit immediately **before** `openspec(<name>): planning`. Find with: `git log --oneline --grep="openspec(<name>): planning" --format="%H" -n 1` then `git rev-parse <hash>~1`.
- **`PREVIOUS_REVIEW_PATH`** = round 1: `(none)`; round ≥ 2: `openspec/changes/<name>/review-r<N-1>.md`.
- **`FOCUS_RANGE`** = round 1: same as `COMMIT_RANGE`; round ≥ 2: `<prev_head>..HEAD`, where `<prev_head>` is the `**Reviewed HEAD:**` value in the previous review file (if that line is missing — a review written before this field existed — fall back to `COMMIT_RANGE`).
- **`RULES_BASE`** = `<baseline_hash>` (the same commit). Reviewers load `REVIEW.md` / `CLAUDE.md` from it, and section 3 classifies depth from it — never from the working tree (`SKILL.md` Context Isolation Rule 7). Load the rules once now: `git show <RULES_BASE>:REVIEW.md`, falling back to `git show <RULES_BASE>:openspec/REVIEW.md`; neither → no project rules.
- **`RULES_TOUCHED`** = whether `git diff --name-only <COMMIT_RANGE>` lists `REVIEW.md`, `openspec/REVIEW.md` or `CLAUDE.md`.
- **`CHECKS_TRIGGERED`** = the ids of the `## Always check` and `## Observe` entries in those rules whose `when:` matches this change: a glob matches a path in `git diff --name-only <COMMIT_RANGE>`, or a keyword appears in an added or removed line of `git diff <COMMIT_RANGE>` (use `git diff <COMMIT_RANGE> | grep -E '^[+-]' | grep -F -- '<keyword>'`). Entries without `when:` count as triggered. This is for the record only (it feeds `## Review stats`, so unused checks can be found later); reviewers do their own matching.
- **`REVIEWED_HEAD`** = `git rev-parse HEAD` now. Phase 5 never commits code, so HEAD must still equal this when `review.md` is written; Phase 6 refuses to archive if it no longer does.

## 2. Full verification gate (machine check before any reviewer)

Read `design.md`'s `## Verification` section and take the **Full** command.

- **No section, or `(none)`** → `{VERIFICATION}` = `(no full verification command defined in design.md)`; continue.
- **Otherwise** → delegate to a **haiku** Agent subagent with the same prompt shape as `phases/04-apply.md` step 1b (opening line, run verbatim from repo root, generous timeout, no file edits, report `exit_code` and the last 40 lines).
  - **exit 0** → `{VERIFICATION}` = `` `<command>` → exit 0 `` + fenced tail. Continue.
  - **non-zero** → do **not** dispatch any reviewer (no point paying for a review of a change that does not build or pass its tests). Write `review.md` directly (section 6) with `Verdict: CHANGES REQUESTED`, Summary `Full verification failed; review skipped until it passes.`, one issue `- [Critical] verification command failed: \`<command>\` exited <n> {class: verification-failure}` followed by the fenced tail, and one suggested revision task `Make \`<command>\` pass (output tail in review.md)`. Go to section 7.

### 2b. Run gate (start the app and drive the scenarios)

Only after Full passed (or is undefined). Take the **Run** line from `design.md`'s `## Verification`.

- **No line, or `(none)…`** → `RUN` = `none: <reason from design.md, or "not defined">`. Append `Run: none — <reason>; scenarios checked by reading only` to `{VERIFICATION}` and continue.
- **Otherwise** → delegate to a **sonnet** Agent subagent (driving an app needs judgment; haiku is not enough). Prompt must contain:
  - Opening: `你是被主 session 委派的子代理，可以直接用 Bash 與其他工具執行，不需要再往下委派。`
  - Job: execute the Run line from the repo root — a shell command verbatim, or for `skill:<name>` invoke that project skill (Skill tool) and follow it. Drive every `#### Scenario:` in `openspec/changes/<name>/specs/**/spec.md` that the Run line covers. Do not edit any tracked file; write evidence only under `$(git rev-parse --git-dir)/super-spec/<name>/run-evidence/round-<N>/` (inside `.git`, so it can never be committed or absorbed by the archive commit).
  - Report exactly: `run_result: PASS | FAIL | ERROR`; a per-step trace (`- <scenario / step> — PASS|FAIL`); evidence paths (screenshots, logs, responses); for the first failed step, the last 40 lines of its log in a fenced block. `ERROR` means the app could not be started at all for an environmental reason (no emulator / device attached, port taken, missing SDK) — say which.

Route on `run_result`:

- **PASS** → append `` `<run line>` → PASS `` + the trace to `{VERIFICATION}`; `RUN` = `PASS (<k> scenarios)`. Continue.
- **FAIL** → same as a failed Full: do **not** dispatch a reviewer; write `review.md` with `Verdict: CHANGES REQUESTED`, Summary `Run verification failed; review skipped until it passes.`, issue `- [Critical] run verification failed at <step>: <one line> {class: run-failure}` + the trace and log tail, suggested revision task `Make <run line> pass (trace in review.md)`. Go to section 7.
- **ERROR** → not the change's fault; do not fail it. `RUN` = `could not run — <reason>`; append that to `{VERIFICATION}` with `scenarios checked by reading only`. Section 7 tells the user which scenarios no one exercised.

## 3. Review depth

Read `## Review Depth` from `proposal.md` (absent → `auto`).

- `Quick` / `Standard` / `Deep` → use it as-is (the user chose it), except for the `RULES_TOUCHED` override below.
- `auto` → classify from the paths the change touches: `git diff --name-only <COMMIT_RANGE> -- . ':(exclude)openspec/'`, using the rules loaded from `RULES_BASE` in section 1 (not the working-tree `REVIEW.md` — the change could have edited it):
  - `RULES_TOUCHED` → **Deep** (a change that edits the review rules gets the strictest review, whatever else it touches)
  - any path matches a glob under `sensitive_paths` → **Deep**
  - every path is documentation or copy (`*.md`, `*.txt`, `*.rst`, `docs/**`, or a glob under `quick_paths`) → **Quick**
  - otherwise → **Standard**
- An explicit `Quick` / `Standard` in `proposal.md` is still overridden to **Deep** when `RULES_TOUCHED` — the proposal is part of the change, so it cannot lower the depth of a change that edits the rules.

Announce the result with its reason, e.g. `Review depth: Deep (matched sensitive path app/src/main/cpp/**)` or `Review depth: Deep (change modifies review rules)`. Depth is a property of what the change touches, not of its size or who wrote it.

## 4. Choose the reviewer path

| Condition | Path |
|---|---|
| Round 1, `## Engine: ultracode`, Workflow tool available, and depth is **Deep** — or depth is **Standard** and the size threshold in `flows/ultracode-review.md` §1 is met | Read `flows/ultracode-review.md` and execute it **instead of** section 5 (it rejoins this file at section 6). |
| Everything else (native engine, Quick depth, round ≥ 2, below threshold, Workflow unavailable) | Section 5: single final-reviewer. |

Round ≥ 2 always takes the single-reviewer path: the focus range is small and the closure check needs the previous review, which the panel does not take.

## 5. Single final-reviewer dispatch

Use the **Agent** tool. Read `prompts/final-reviewer.md` and substitute:
- `{CHANGE_NAME}`: `<name>`
- `{PROPOSAL_PATH}`: `openspec/changes/<name>/proposal.md`
- `{DESIGN_PATH}`: `openspec/changes/<name>/design.md`
- `{TASKS_PATH}`: `openspec/changes/<name>/tasks.md`
- `{SPECS_DIR}`: `openspec/changes/<name>/specs/`
- `{COMMIT_RANGE}`, `{REVIEW_ROUND}` (= N), `{PREVIOUS_REVIEW_PATH}`, `{FOCUS_RANGE}`: from section 1
- `{VERIFICATION}`: from section 2
- `{REVIEW_DEPTH}`: from section 3
- `{RULES_BASE}`: from section 1

**Model:** `sonnet` when depth is `Quick`; otherwise `opus`.

**Forbidden:** per-task reviewer reports; implementer narratives; the brainstorming conversation. (The previous round's review file is allowed — the prompt reads it via `{PREVIOUS_REVIEW_PATH}`.)

The prompt instructs the reviewer to invoke `/code-review` (and, for Standard / Deep, `/security-review`) report-only.

The reviewer returns one of:
- `VERDICT: APPROVED` + summary + optional `## Closure`, `## Notes` (`[Minor]`), `## Pre-existing`
- `VERDICT: CHANGES REQUESTED` + severity-tagged issues + suggested revision tasks
- `VERDICT: NEEDS DESIGN UPDATE` + severity-tagged design issues + suggested resolutions

### Severity rule (the reviewer applies this; orchestrator re-validates)

- `APPROVED` ⇔ zero `[Critical]` and zero `[Important]` issues. `[Minor]` items belong under `## Notes`, `[Pre-existing]` under `## Pre-existing`.
- `CHANGES REQUESTED` ⇔ at least one `[Critical]` or `[Important]` issue, design itself is sound.
- `NEEDS DESIGN UPDATE` ⇔ design itself is flawed.
- Every `[Critical]` / `[Important]` line ends with `{class: …}`.

If the reviewer's output contradicts this rule (e.g., `APPROVED` while listing `[Critical]` issues, `CHANGES REQUESTED` with only `[Minor]` / `[Pre-existing]` items, or a blocking line without a class), **do not silently rewrite it**: re-dispatch the final-reviewer once with a note pointing out the inconsistency, then go with the corrected output.

### Deep depth → independent verification

If depth is **Deep** and the (re-validated) verdict is not `APPROVED`, Read `flows/native-verify.md` and execute it before writing `review.md`: two independent skeptics vote on each blocking issue and the verdict is recomputed from the votes, exactly like the ultracode panel does.

## 6. Write `review.md`

Path: `openspec/changes/<name>/review.md`

Structure: see `templates/review.md`. Fill in the header fields (`Round`, `Reviewed range` = `COMMIT_RANGE`, `Focus range`, `Reviewed HEAD` = `REVIEWED_HEAD`, `Review depth`, `Review rules` = `RULES_BASE` + `RULES_TOUCHED`, `Verification`, `Run`), then `Verdict`, `Summary`, and the body sections from the reviewer output (or the panel / native-verify result).

Fill `## Review stats` from `tasks.md` (the `**Review rounds:** … · **Verification failures:** …` line under each Task, written in Phase 4):
- Tasks: total Task headers (including revision tasks), and how many were skipped
- Passed review in one round: count of Tasks with `Review rounds: 1` and `Verification failures: 0`, as `k / n (pct%)`
- Average and maximum review rounds per Task
- Total verification failures in Phase 4
- Final review round: `N`
- Panel / skeptic numbers when they ran: candidates, verified, downgraded, moved to pre-existing
- Pre-existing items found (tasks.md + this review)
- Checks triggered: `CHECKS_TRIGGERED` (ids, or `none`), and Observe hits: the `observe:<id>` notes the reviewers reported

Tasks without the stats line (applied before this field existed) are excluded from the averages; say so in one line.

Write `review.md` to disk but **do NOT commit it here**. It stays uncommitted in the working tree; Phase 6's archive-committer absorbs it into the single squashed archive commit alongside the archived file moves. This keeps the branch from accumulating a stand-alone `openspec(<name>): final review` scaffolding commit when no further code changes follow.

(Exception — `CHANGES REQUESTED` path: the `openspec(<name>): revision N tasks` commit described below carries the renamed `review-r<N>.md` with it, because it precedes new feat work and acts as the planning commit for the revision round.)

## 7. Route by verdict

### APPROVED

1. **Dispositions.** If the review carries any item a skeptic downgraded (marked `skeptic 投票自 … 降級` in Notes) or any `## Accepted open issues`, every one of them needs an answer before archive. Ask with AskUserQuestion — up to 4 items per call, one question per item, options:
   - **False positive** — the reviewer was wrong
   - **Intentional** — the behavior is deliberate
   - **Not important** — real but not worth fixing
   - **Fix later** — real; goes to the project backlog at archive time

   Record the answers under `## Dispositions` in `review.md` (`- <item> → <answer>`). Plain `[Minor]` notes from the reviewer do not need a disposition.
2. **Todo:** mark `Phase 5 · Final review` `completed`; leave `Phase 6 · Archive` `pending` (see `flows/todo-tracking.md`).
3. Show the user the verdict, Notes, Pre-existing items and the `## Review stats` block. Always show the `Run` line; when it is not `PASS`, say plainly that no one started the app for this change. If depth was **Deep**, also show the `Verification` line and say plainly which scenarios the review could not exercise (from any `cannot-determine` notes, and every scenario the Run did not cover) — those are what a human should look at before saying "archive".
4. Wait for the user's explicit "archive" instruction. Do NOT auto-archive.

### CHANGES REQUESTED or NEEDS DESIGN UPDATE — round cap first

If **N ≥ 3**, do not start another revision round automatically. Three review rounds without converging usually means the approach is wrong, not the code. Show the open issues and ask (AskUserQuestion):

- **Revisit the design** → Read `flows/recover.md` (works for either verdict).
- **Accept the remaining issues** → ask for one line of reasoning per open issue (free text). Rewrite `review.md`: `Verdict: APPROVED`; move every remaining `[Critical]` / `[Important]` line into `## Accepted open issues` as `- <issue> — accepted: <reason>`; prefix Summary with `Approved by user at the round-3 cap with open issues.` Then continue with the APPROVED route above (its dispositions step will cover these).
- **Intervene manually** → pause; you fix things, then resume with `/super-spec <name>` (Phase 5 runs again as round N+1, past the cap).

Otherwise (N < 3), route by verdict:

- **CHANGES REQUESTED** → Use AskUserQuestion: "Reviewer requested changes. Add revision tasks now?"
  - `yes` →
    1. Rename `review.md` → `review-r<N>.md` (the next round reads it via `{PREVIOUS_REVIEW_PATH}`).
    2. Append `### Revision N - Task M:` blocks to `tasks.md` addressing every `[Critical]` and `[Important]` issue (omit `[Minor]` / `[Pre-existing]`; they're recorded but not gating). Each block names the issue's class and says **fix every instance of `<class>`**, and lists the affected files under `**Files:**`.
    3. Commit `tasks.md` and `review-r<N>.md` together: `openspec(<name>): revision N tasks`.
    4. Loop back to Phase 4 for the new tasks only. **Todo:** append a `Phase 4 · Revision N Task M: <title>` item (`pending`) per new Task and set `Phase 5 · Final review` back to `pending`.
  - `no` → pause; user can resume later
- **NEEDS DESIGN UPDATE** → Jump to `flows/recover.md`
