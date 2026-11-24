# Escape — record a bug that got past review

**Trigger:** the user reports that an **archived** change shipped a bug the reviews should have caught ("the change X broke Y", "review missed this", "record an escape"). Also offered at the end of any `/super-spec` change whose purpose is fixing a bug introduced by an earlier archived change.

**Purpose:** every escape becomes a regression case for the reviewers. When the reviewer prompts, lenses, or model table change, they are re-scored against these cases (see `evals/README.md` in this skill). A prompt change that stops catching a recorded escape is rolled back.

This flow writes the case file (plus any `REVIEW.md` / backlog lines step 4 adds) and makes one commit. It never changes code.

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
- missing_layer: <one line: the strongest layer that should have caught it — structure, type, lint, test, Run / verify skill, sensitive path, or a REVIEW.md rule>

## What goes wrong
<1-2 sentences, confirmed by the user>
```

## 4. Close the loop at the source

**Read `flows/enforcement-ladder.md` and execute it** for this case's class, with the escape case file as the evidence and as `prove_red_on`. An escape is the strongest evidence a class can have — a real bug, at a known commit and line — so the ladder should reach level 1–4 whenever the class is mechanically detectable; a text rule alone is the fallback, not the default.

Independently of the ladder, offer (AskUserQuestion, multiSelect) the fixes `missing_layer` points at that the ladder does not cover:
- add the path to `REVIEW.md` → `sensitive_paths`
- extend `## Verification` (the Run line, or the verify skill's feature map / Doctor) — note it as a backlog item in `openspec/backlog.md`

**A level-5 check must survive a skeptic before it is written.** Draft it in the trigger format — `- [<next free id>] when: <globs / keywords that would have matched this escape's diff> → check: <what to look for> (source: escape <case file name>) · enforced_by: reviewer` — then dispatch one **sonnet** Agent subagent with `prompts/rule-skeptic.md`: `{RULE_LINE}` = the draft line, `{RULE_EVIDENCE}` = the case file path, the change's archive commit and the bug's file:lines. Act on the JSON it returns:
- `refuted: false` → write the line under `## Observe` in `REVIEW.md` (create the section from `templates/REVIEW-rules.md` if missing). It reports non-blocking `observe:` hits until you promote it to `## Always check`.
- `refuted: true` → do not write it. Show the reasoning, and the `better_trigger` if one was given; offer once to retry with that trigger (one more skeptic). Otherwise record `missing_layer` as is — the case still counts in evals.

Add one line to the case file: `- enforcement: <level> <enforced_by value> | none — <reason>`.

## 5. Commit

Delegate to a haiku Agent subagent (standard opening line; `git add` only the files written — the case file, and `REVIEW.md` / `openspec/backlog.md` if step 4 changed them; no hook bypass; report stderr on failure): `docs(review-evals): record escape from <name>`.
