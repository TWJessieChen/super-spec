# Escape — record a bug that got past review

**Trigger:** the user reports that an **archived** change shipped a bug the reviews should have caught ("the change X broke Y", "review missed this", "record an escape"). Also offered at the end of any `/super-spec` change whose purpose is fixing a bug introduced by an earlier archived change.

**Purpose:** every escape becomes a regression case for the reviewers. When the reviewer prompts, lenses, or model table change, they are re-scored against these cases (see `evals/README.md` in this skill). A prompt change that stops catching a recorded escape is rolled back.

This flow writes one file and makes one commit. It never changes code.

## 1. Pin down the escape

Ask only for what you cannot find yourself:

1. **Which change.** Find it under `openspec/changes/archive/*-<name>/`. Its archive commit is `git log --format=%H -n 1 -- openspec/changes/archive/*-<name>/proposal.md`.
2. **Where the bug is.** File and line(s) in that change's diff — confirm with `git show <archive_commit> -- <file>` that those lines were added or modified by the change. If they were not, it is not an escape of this change; say so and stop.
3. **What goes wrong.** One or two sentences of observable behavior.
4. **The fix**, if one exists: commit hash (or "not fixed yet").
5. **Class**: a kebab-case name for the kind of mistake, matching existing classes in the archive when one fits (`git grep -h "class: " openspec/changes/archive | sort | uniq -c`).

Ask the user to confirm the location and description before writing — the case is only useful if a person who knows the code agrees that is the bug.

## 2. Which layer should have caught it

Read the archived `review*.md` and `tasks.md` of that change and record, in one line each:
- **Review depth used**, and whether the path should have been in `sensitive_paths`.
- **Verification**: did a command run, and could a test have caught this?
- **Was it seen?** Search the archived reviews for the file — was the issue reported and then downgraded by skeptics, accepted at the round cap, given a disposition, or never mentioned?

## 3. Write the case

Write `openspec/review-evals/<YYYY-MM-DD>-<name>-<class>.md`:

```
# Escape: <one-line description>

- change: <name>
- archive_commit: <hash>
- range: <baseline>..<last implementation commit before the archive squash, or the archive commit itself>
- file: <path>
- lines: <start>-<end>
- class: <kebab-case>
- expected_severity: <Critical | Important>
- fix_commit: <hash | none>
- review_depth_used: <Quick | Standard | Deep>
- seen_by_review: <never | downgraded | accepted | disposed:<answer>>
- missing_layer: <one line: what should have caught it — a REVIEW.md rule, a sensitive path, a test, a verification command>

## What goes wrong
<1-2 sentences, confirmed by the user>
```

## 4. Close the loop at the source

Based on `missing_layer`, offer (AskUserQuestion, multiSelect) the fixes that apply:
- add a line to `REVIEW.md` → `## Always check`
- add the path to `REVIEW.md` → `sensitive_paths`
- add a test or extend `## Verification` — note it as a backlog item in `openspec/backlog.md`

## 5. Commit

Delegate to a haiku Agent subagent (standard opening line; `git add` only the files written; no hook bypass; report stderr on failure): `docs(review-evals): record escape from <name>`.
