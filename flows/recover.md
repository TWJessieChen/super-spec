# Recover (NEEDS DESIGN UPDATE)

Triggered when the final-reviewer (Phase 5) returns `VERDICT: NEEDS DESIGN UPDATE`.

## Steps

1. Append to `openspec/changes/<name>/design.md` per `templates/revision-block.md`. (N = next integer; first revision = 1.)

2. Commit: `openspec(<name>): design revision N`.

3. Re-enter Phase 3 to regenerate `tasks.md`. Existing `- [x]` Tasks remain. Append `### Revision N - Task M:` blocks for the new work.

4. Commit: `openspec(<name>): revision N tasks`.

5. Re-enter Phase 4 to apply only the new Tasks.

6. Re-enter Phase 5.
