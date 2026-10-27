# Recover (design revision)

Triggered when:
- the final-reviewer (Phase 5) returns `VERDICT: NEEDS DESIGN UPDATE`, or
- the user picks **Revisit the design** at a round cap — the Phase 4 three-FAIL cap on one Task, or the Phase 5 round-3 cap.

The reviewer issues that go into the revision block are, respectively: the `## Design Issues` of `review.md`; the blocking issues of the Task's last failed round; or the open `[Critical]` / `[Important]` issues of `review.md`.

## Steps

> **Todo:** this flow adds a revision round. When step 5 regenerates `tasks.md` with `### Revision N - Task M:` blocks, append a `Phase 4 · Revision N Task M: <title>` item (`pending`) per new Task and set `Phase 5 · Final review` back to `pending` (see `flows/todo-tracking.md`).

1. If this flow came from Phase 5, rename `openspec/changes/<name>/review.md` → `review-r<N>.md` (N = its `**Round:**`), so the next final review runs as round N+1 and can check closure against it.

2. Append to `openspec/changes/<name>/design.md` per `templates/revision-block.md`. (Revision number = next integer; first revision = 1.) List each issue with its `{class: …}`.

3. If the revision changes requirements, update the affected `specs/<capability>/spec.md` deltas to match (same structure as `templates/spec-delta.md`). If it changes how the change is verified, update `design.md`'s `## Verification` too.

4. Commit `design.md`, any changed spec deltas, and the renamed `review-r<N>.md` (if any): `openspec(<name>): design revision N`.

5. Re-enter Phase 3 to regenerate `tasks.md`. Existing `- [x]` Tasks remain. Append `### Revision N - Task M:` blocks for the new work. When this flow came from the Phase 4 cap, the stuck Task's `- [ ]` block is replaced by its revision blocks (mark the original `- [x]` with `（superseded by Revision N）`).

6. Commit: `openspec(<name>): revision N tasks`.

7. Re-enter Phase 4 to apply only the new Tasks.

8. Re-enter Phase 5.
