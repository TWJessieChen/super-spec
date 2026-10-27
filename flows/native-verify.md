# Native Verify — skeptic voting outside the Workflow engine

**Trigger:** Phase 5 took the single-reviewer path (section 5 of `phases/05-final-review.md`), depth is **Deep**, and the re-validated verdict is `CHANGES REQUESTED` or `NEEDS DESIGN UPDATE`.

**Purpose:** a Deep change gets the same false-positive filter whether or not it runs on the ultracode engine. Each blocking finding is checked by two independent skeptics; only findings that survive keep blocking. This mirrors the Verify phase of `workflows/final-review.js` — same prompt, same vote rules.

Read-only: nothing here writes to git.

## 1. Collect candidates

From the reviewer output, take every `[Critical]` / `[Important]` line under `## Issues` or `## Design Issues`. Mark lines under `## Design Issues` as `is_design_flaw = true`.

Sort Critical first. Verify at most **12**; any beyond that keep their original severity unverified (announce: `N blocking issues exceed the skeptic limit of 12; the rest stay blocking without verification`).

## 2. Dispatch skeptics (all in one message, in parallel)

For each candidate, dispatch **two** Agent subagents (`model: sonnet`). Prompt = the full text of `prompts/_isolation-preamble.md`, a `---` line, then `prompts/final-skeptic.md` with:
- `{ISSUE_SEVERITY}`: `Critical` or `Important`
- `{ISSUE_FILE}`: the file path from the issue line (text before the first ` — `, without `:line`)
- `{ISSUE_DESCRIPTION}`: the whole issue line without its severity tag
- `{DESIGN_PATH}`: `openspec/changes/<name>/design.md`
- `{COMMIT_RANGE}`: the Phase 5 `COMMIT_RANGE`

Each skeptic answers with one JSON line `{"refuted": bool, "pre_existing": bool, "reasoning": "…"}`. An answer that does not parse counts as no vote.

## 3. Settle each candidate

`confirms` = number of valid votes with `refuted: false`; `preVotes` = valid votes with `refuted: true` and `pre_existing: true`.

| Votes | Result |
|---|---|
| `confirms ≥ 2` | keeps its severity |
| `confirms = 1` | drops one level (Critical → Important, Important → Minor) |
| `confirms = 0`, `preVotes ≥ 1` | becomes `[Pre-existing]` |
| `confirms = 0` otherwise | becomes `[Minor]` |

## 4. Recompute the verdict and rewrite the reviewer output

- Any surviving `[Critical]` / `[Important]` with `is_design_flaw` → `NEEDS DESIGN UPDATE`
- Else any surviving `[Critical]` / `[Important]` → `CHANGES REQUESTED`
- Else → `APPROVED`

Then rewrite the reviewer output you will put into `review.md`:
- Issues that dropped to `[Minor]` move to `## Notes` with the suffix `（skeptic 投票自 [<original>] 降級，confirms=<n>——請人工複核）`. These need a disposition before archive (Phase 5 section 7).
- Issues that became `[Pre-existing]` move to `## Pre-existing` with `（原標 [<original>]，skeptic 判定早於本 change）`.
- Drop the `## Suggested revision tasks` / `## Suggested resolutions` lines that belonged to issues that no longer block. If you cannot tell which line belongs to which issue, keep them all and add a note saying so.
- If the verdict changed to `APPROVED`, keep the reviewer's Summary and append: `Independent verification did not confirm the blocking findings; see Notes.`
- Record the vote counts for `## Review stats` (candidates, verified, downgraded, moved to pre-existing).

Do not re-dispatch the final-reviewer. Return to `phases/05-final-review.md` section 6.
