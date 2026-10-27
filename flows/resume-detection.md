# Resume Detection

Run: `openspec status --change <name> --json`. Parse, then route by detected state:

| State | Jump to |
|---|---|
| no `design.md` | Phase 1 |
| `design.md` exists, no `specs/` delta files | Phase 2 (resume at "Write spec deltas"; skip HARD-GATE B if `proposal.md` already has `## Mode`) |
| `design.md` exists, no `tasks.md` | Phase 3 |
| `tasks.md` exists with `- [ ]` items | Phase 4 |
| all tasks `- [x]`, no `review.md` | Phase 5 (round = 1 + number of `review-r*.md` files) |
| `review.md` verdict = `APPROVED`, not yet archived | Phase 6 |
| `review.md` verdict = `CHANGES REQUESTED` (the user answered `no` to revision tasks, or paused at the round cap) | Phase 5 section 7 — re-ask the round-cap / revision-tasks question for that review; do not re-review |
| `review.md` verdict = `NEEDS DESIGN UPDATE` | Recover flow (`flows/recover.md`) |

Announce: `Resuming <name> at <phase>.`

## Phase-entry hooks still run on resume

Resuming into a phase means **entering that phase** — not skipping past its entry steps. Notably:

- Resuming into Phase 4 runs the **Phase-4 entry sweep** (`flows/phase4-sweep.md`). The sweep is idempotent: a no-op when no superpowers artifacts/commits remain, and the right thing to do otherwise.
- Resuming into Phase 4 also re-runs the **Engine routing** step: if `proposal.md` has `## Engine: ultracode`, follow `flows/ultracode-apply.md` after the sweep. Cross-session there is **no** `resumeFromRunId` (it is same-session-only) — launch a **fresh** Workflow run with args rebuilt from the remaining `- [ ]` Tasks in `tasks.md`. This is always safe: git + tasks.md are the truth source; only the journal cache is lost.
- Resuming into Phase 5 dispatches the final-reviewer fresh (the prior dispatch's output is not preserved). The round number and focus range are recomputed from the `review-r*.md` files on disk, so an interrupted round 2 resumes as round 2, not as a fresh whole-change review.

Do not try to be clever and skip the entry steps "because we already ran them last time". Re-run them.

## `review.md` may be uncommitted at Phase 6 resume

By design, Phase 5 leaves `review.md` uncommitted (it is folded into the archive commit). When `openspec status` routes a resume to Phase 6 ("`review.md` verdict = `APPROVED`, not yet archived"), the file may exist only in the working tree, not yet in git history. This is expected — proceed into Phase 6 normally. Phase 6 still checks that HEAD equals the review's `**Reviewed HEAD:**`; commits made between sessions are exactly what that check catches.
