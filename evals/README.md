# Reviewer evals

super-spec's reviewers are prompts plus a model table. Changing either can make reviews better or quietly worse. This folder describes how to tell which, using bugs that really escaped review as the test set.

## Where cases live

Cases are recorded **in each project that uses super-spec**, not in this repo, because they point at that project's commits:

```
<project>/openspec/review-evals/<date>-<change>-<class>.md
```

`flows/escape.md` writes them. Each case names a commit range, the file and lines of a confirmed bug that range introduced, the bug's class, and the severity a reviewer should have given it.

You can also add **clean cases**: a range from an archived change that has been in use for a while with no reported bug. Write the same file with `expected_severity: none` and no `file` / `lines`. Clean cases measure noise: a reviewer that blocks a clean range is producing false positives.

## Red proof is the same idea for checks

An escape case is also the test set for the *checks* it led to. When `flows/enforcement-ladder.md` turns a class into a lint, test or type, the backlog item's `prove_red_on` points at a case file (or a real past occurrence), and the change that builds the check must show it failing there (`design.md` `Red proof:`, run by Phase 5 §2c). A reviewer prompt is kept only if it still catches the recorded escapes; a check is kept only if it fails on them.

## When to run

Before merging any change to this repo that touches:
- `prompts/task-reviewer.md`, `prompts/final-reviewer.md`, `prompts/final-lens-*.md`, `prompts/final-skeptic.md`, `prompts/_isolation-preamble.md` (it defines how reviewers load and apply `REVIEW.md`, including `when:` triggers and `## Observe`)
- the vote rules in `workflows/final-review.js` or `flows/native-verify.md`
- the model assignments in `phases/02-propose.md` / `templates/proposal.md`

## How to run (manual, per case)

1. In the project, check out the case's range end in a scratch worktree: `git worktree add ../eval-<case> <range end>`.
2. Dispatch the reviewer under test exactly as the workflow would — `prompts/final-reviewer.md` with `{COMMIT_RANGE}` = the case's range, `{REVIEW_ROUND}` = 1, `{FOCUS_RANGE}` = the range, `{REVIEW_DEPTH}` = `Standard` (or the depth the case says should have applied), `{VERIFICATION}` = `(no full verification command defined in design.md)`, `{RULES_BASE}` = the case's range start (so the reviewer sees the project rules as they were when that change was reviewed).
3. Score:
   - **Escape case** — *caught* if any `[Critical]` / `[Important]` issue cites the case's file within ±5 lines of its range, or names its class. Caught at a lower severity counts as *seen*, not caught.
   - **Clean case** — *false positive* for every `[Critical]` / `[Important]` issue.
   - **When skeptics ran** (Deep depth via `flows/native-verify.md`, or the ultracode panel via `workflows/final-review.js`), score the escape case twice: **before the vote** (from the reviewer's / lenses' raw blocking candidates — the `## Skeptic ledger` lines) and **after the vote** (the final `[Critical]` / `[Important]` issues). An escape that is *caught before* but *not after* was voted down: the skeptic step removed a real bug.
4. Remove the worktree.

Record per run: `caught / escapes`, `seen / escapes`, `false positives / clean cases`, and the reviewer prompt's git hash — and, for skeptic runs, `caught before vote / escapes` next to `caught after vote / escapes`.

**Reading the two recall numbers.** If `caught after` is well below `caught before`, the vote rules (or `prompts/final-skeptic.md`) are too eager to refute; fix that before touching the reviewer prompts, because a better reviewer cannot help if its finds are voted away. If both are low, the reviewer itself is missing the bug. Stop tuning for recall when the next gain would only come from matching these specific cases (wording tailored to one file or class) rather than catching a kind of bug — that is overfitting the eval set. Keep the cases you tune against separate from a few held-out cases you only score.

## Keep-or-revert rule

Run the old and the new prompt on the same cases. Keep the change only if `caught` does not drop and false positives do not rise; if one improves and the other gets worse, the change needs a stated reason in its commit message. The priority order, as in the review itself, is: catch real bugs first, reduce noise second.

## Live signal from normal use

Besides the eval set, every archived `review.md` carries data for the same questions:
- `## Review stats` — rounds per Task and one-round pass rate (does review converge?)
- `## Dispositions` — how often a downgraded finding was a false positive versus real (is the skeptic filter calibrated?)
- `## Skeptic ledger` — every blocking candidate before the vote and what the vote did to it; join it with recorded escapes to see whether a voted-down finding later shipped as a bug
- `Checks triggered` / `Observe hits` (in `## Review stats`) — which `REVIEW.md` checks fired, and what trial checks would have flagged

Collect them with `git grep -h -A10 "## Review stats" -- 'openspec/changes/archive/*/review.md'` and `git grep -h -A20 "## Skeptic ledger" -- 'openspec/changes/archive/*/review.md'`.

Not done yet (needs about 10 archived reviews first): pruning `REVIEW.md` checks that never trigger or never find a real bug. See the README section "Verification upgrade (2026-11)".
