# Phase 5 — Final Review

Dispatch the final-reviewer subagent.

Use the **Agent** tool. Read `prompts/final-reviewer.md` and substitute:
- `{CHANGE_NAME}`: `<name>`
- `{PROPOSAL_PATH}`: `openspec/changes/<name>/proposal.md`
- `{DESIGN_PATH}`: `openspec/changes/<name>/design.md`
- `{TASKS_PATH}`: `openspec/changes/<name>/tasks.md`
- `{COMMIT_RANGE}`: `<baseline_hash>..HEAD` where baseline = the commit immediately **before** `openspec(<name>): propose`. Find with: `git log --oneline --grep="openspec(<name>): propose" --format="%H" -n 1` then `git rev-parse <hash>~1`.

**Model:** `claude-opus-4-7`

**Forbidden:** per-task reviewer reports; implementer narratives; the brainstorming conversation.

The prompt instructs the reviewer to invoke `/simplify` skill internally for cross-task DRY analysis.

The reviewer returns one of:
- `VERDICT: APPROVED` + summary
- `VERDICT: CHANGES REQUESTED` + issues + suggested revision tasks
- `VERDICT: NEEDS DESIGN UPDATE` + design issues + suggested resolutions

## Write `review.md`

Path: `openspec/changes/<name>/review.md`

Structure: see `templates/review.md`. Fill in `Verdict`, `Summary` (from reviewer), `Issues` (bullet list, empty if APPROVED).

Stage `review.md`, commit message: `openspec(<name>): final review`.

## Route by verdict

- **APPROVED** → Show user the verdict. Wait for the user's explicit "archive" instruction. Do NOT auto-archive.
- **CHANGES REQUESTED** → Use AskUserQuestion: "Reviewer requested changes. Add revision tasks now?"
  - `yes` → append `### Revision N - Task M:` blocks to `tasks.md` addressing each issue, commit `openspec(<name>): revision N tasks`, then loop back to Phase 4 for the new tasks only
  - `no` → pause; user can resume later
- **NEEDS DESIGN UPDATE** → Jump to `flows/recover.md`
