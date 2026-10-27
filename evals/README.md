# Reviewer evals

super-spec's reviewers are prompts plus a model table. Changing either can make reviews better or quietly worse. This folder describes how to tell which, using bugs that really escaped review as the test set.

## Where cases live

Cases are recorded **in each project that uses super-spec**, not in this repo, because they point at that project's commits:

```
<project>/openspec/review-evals/<date>-<change>-<class>.md
```

`flows/escape.md` writes them. Each case names a commit range, the file and lines of a confirmed bug that range introduced, the bug's class, and the severity a reviewer should have given it.

You can also add **clean cases**: a range from an archived change that has been in use for a while with no reported bug. Write the same file with `expected_severity: none` and no `file` / `lines`. Clean cases measure noise: a reviewer that blocks a clean range is producing false positives.

## When to run

Before merging any change to this repo that touches:
- `prompts/task-reviewer.md`, `prompts/final-reviewer.md`, `prompts/final-lens-*.md`, `prompts/final-skeptic.md`, `prompts/_isolation-preamble.md`
- the vote rules in `workflows/final-review.js` or `flows/native-verify.md`
- the model assignments in `phases/02-propose.md` / `templates/proposal.md`

## How to run (manual, per case)

1. In the project, check out the case's range end in a scratch worktree: `git worktree add ../eval-<case> <range end>`.
2. Dispatch the reviewer under test exactly as the workflow would — `prompts/final-reviewer.md` with `{COMMIT_RANGE}` = the case's range, `{REVIEW_ROUND}` = 1, `{FOCUS_RANGE}` = the range, `{REVIEW_DEPTH}` = `Standard` (or the depth the case says should have applied), `{VERIFICATION}` = `(no full verification command defined in design.md)`.
3. Score:
   - **Escape case** — *caught* if any `[Critical]` / `[Important]` issue cites the case's file within ±5 lines of its range, or names its class. Caught at a lower severity counts as *seen*, not caught.
   - **Clean case** — *false positive* for every `[Critical]` / `[Important]` issue.
4. Remove the worktree.

Record per run: `caught / escapes`, `seen / escapes`, `false positives / clean cases`, and the reviewer prompt's git hash.

## Keep-or-revert rule

Run the old and the new prompt on the same cases. Keep the change only if `caught` does not drop and false positives do not rise; if one improves and the other gets worse, the change needs a stated reason in its commit message. The priority order, as in the review itself, is: catch real bugs first, reduce noise second.

## Live signal from normal use

Besides the eval set, every archived `review.md` carries data for the same two questions:
- `## Review stats` — rounds per Task and one-round pass rate (does review converge?)
- `## Dispositions` — how often a downgraded finding was a false positive versus real (is the skeptic filter calibrated?)

Collect them with `git grep -h -A8 "## Review stats" -- 'openspec/changes/archive/*/review.md'`.
