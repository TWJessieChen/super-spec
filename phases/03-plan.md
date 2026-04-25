# Phase 3 — Plan

Invoke `superpowers:writing-plans` via the Skill tool. In the invocation, instruct:

- Write the plan to `openspec/changes/<name>/tasks.md` (override the default `docs/superpowers/plans/` path)
- Read the design from `openspec/changes/<name>/design.md`
- Read the mode from `openspec/changes/<name>/proposal.md`'s `## Mode` section
- For **TDD** mode: each Task includes explicit "write failing test → run to confirm fail → implement minimal code → run to confirm pass → commit" sub-steps
- For **Simple** mode: omit test-writing sub-steps; include "verify existing tests still pass" as a final sub-step in each Task

Each Task is a unit of subagent work (~15min – 2hr). Sub-steps within a Task are 2–5 minute atomic actions with `- [ ]` checkboxes.

When `tasks.md` is written, commit:
- Stage `openspec/changes/<name>/tasks.md`
- Commit message: `openspec(<name>): plan`

→ Continue to Phase 4.
