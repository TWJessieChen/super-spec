---
name: super-spec
description: Spec-driven workflow combining superpowers (brainstorm/plan/subagent/TDD/review) with openspec (lifecycle). Triggered by /super-spec, or when user wants a structured multi-phase workflow for a non-trivial change with subagent isolation, per-task review, and final holistic review before archive.
---

# /super-spec

Single-entry workflow for a non-trivial change. End-to-end:

```
brainstorm → propose → plan → apply (per-task subagent + verification gate + spec/quality review) → final review (rounds, depth by risk) → archive
```

All artifacts live in `openspec/changes/<name>/`. Defaults: TDD mode is offered, no merge / no push / no PR.

---

## ⚠ EXECUTION MODEL

This file is an **index only** — all phase, flow, and template content lives in sibling files. You MUST Read each sibling file at the moment you enter its step, every invocation, with **no skipping, paraphrasing, or inferring contents from file names**. If a Read fails, halt and report the path; do not improvise. A correct run produces a long sequence of Read calls.

---

## Invocation

User runs `/super-spec [--ultra] [<name | short description>]`.

- **`--ultra` flag** (optional, position-independent): pre-answers Phase 2's HARD-GATE B2 with the `ultracode` engine — the gate still shows the agent-model table for confirmation; it is never silently skipped. Strip the flag before interpreting the remaining argument as `<name | description>`.

1. **Read `flows/pre-flight.md` and execute it.** Halt on any failure.
2. Then determine the **entry phase** based on argument:
   - **No argument** → ask: "What change do you want to work on? Describe what to build or fix." (entry phase = Phase 1 once they answer)
   - **`<name>` matches existing `openspec/changes/<name>/`** → **Read `flows/resume-detection.md` and execute it** to determine the entry phase.
   - **`<name>` is new, or only a description was given** → entry phase = Phase 1.
3. **Read `flows/todo-tracking.md` and initialize the visualized todo list** for the determined entry phase. Then enter that phase (Read its `phases/0N-*.md`).

---

## Phase index

Phases run sequentially unless a flow directs otherwise. Each row is an instruction: when you enter the phase, Read the file and execute it line-by-line.

| Phase | Action |
|---|---|
| 1. Brainstorm | Read `phases/01-brainstorm.md`. |
| 2. Propose | Read `phases/02-propose.md`. |
| 3. Plan | Read `phases/03-plan.md`. |
| 4. Apply (per-Task loop) | Read `phases/04-apply.md`. Re-Read at the top of every Task iteration — do not loop from memory. |
| 5. Final Review | Read `phases/05-final-review.md`. |
| 6. Archive | Read `phases/06-archive.md`. |

## Flow index (non-linear)

Enter only when the trigger fires; Read the file and execute it.

| Trigger | Action |
|---|---|
| Every invocation start (mandatory) | Read `flows/pre-flight.md`. |
| Entry phase determined — initialize/refresh the visualized todo list (mandatory) | Read `flows/todo-tracking.md`. |
| Resuming an existing change | Read `flows/resume-detection.md`. |
| Phase 4 entry (once, both engines — before Engine routing) | Read `flows/phase4-sweep.md`. |
| Phase 4 entry with `proposal.md` `## Engine: ultracode` (after the entry sweep) | Read `flows/ultracode-apply.md`. |
| Phase 5 section 4 selects the panel (round 1, `## Engine: ultracode`, depth Deep — or Standard above the size threshold) | Read `flows/ultracode-review.md`. |
| Phase 5 single-reviewer path, depth Deep, verdict not APPROVED | Read `flows/native-verify.md`. |
| Phase 5 verdict = `NEEDS DESIGN UPDATE`, or the user picks "Revisit the design" at a round cap (Phase 4 or Phase 5) | Read `flows/recover.md`. |
| User says "abort" / "stop this change" | Read `flows/abort.md`. |
| User reports a bug that an archived change shipped and review missed ("record an escape") | Read `flows/escape.md`. |

## Template index

When a phase file tells you to write an artifact, Read the matching template first for its exact structure — do not invent.

| Artifact | Template |
|---|---|
| `proposal.md` | `templates/proposal.md` |
| `design.md` | `templates/design.md` |
| `specs/<capability>/spec.md` (spec delta) | `templates/spec-delta.md` |
| `review.md` | `templates/review.md` |
| `## Revisions` block (Recover flow) | `templates/revision-block.md` |
| `REVIEW.md` (project review rules; offered by pre-flight) | `templates/REVIEW.md`, or `examples/REVIEW.android.md` |

---

## Context Isolation Rules (NON-NEGOTIABLE)

These deliver the actual isolation between phases. Apply to every Phase-4 dispatch and to Phase 5.

1. **Subagent dispatch always uses the Agent tool, never Skill.** Skill runs in the current context (no isolation); Agent spawns a fresh context. Skill is only for invoking other skills' interactive logic inside the orchestrator (e.g., `superpowers:brainstorming`, `superpowers:writing-plans`, `/code-review` inside the final-reviewer).
2. **Build every subagent prompt from `prompts/*.md`.** Read at dispatch time, substitute variables, pass verbatim. No freehand. No extra context unless a `{...}` placeholder invites it. The shared `prompts/_isolation-preamble.md` is referenced by every per-role prompt — do not strip it.
3. **Reviewers see git, not implementer narrative.** Implementer returns `Done: <commit_hash>`; reviewer prompts contain the hash and fetch the diff themselves. Two things may be added, both through template placeholders: the **same review's previous-round issue list** (so re-reviews check closure and look only at what changed) and **machine verification output** the orchestrator produced itself. Never paste implementer narrative into a reviewer prompt.
4. **Per-task fresh subagent.** Never reuse a subagent across Tasks — fresh implementer + reviewer per dispatch.
5. **Strip orchestrator context.** Brainstorming conversation, prior Task outputs, reviewer reports — none of these go into a subagent prompt unless the template has a placeholder for them.
6. **Final-reviewer is also fresh.** Reads files from disk and runs git itself; receives no per-task summaries.

---

## Review convergence rules (NON-NEGOTIABLE)

Review exists to catch real bugs without trapping an agent author in endless rounds.

1. **Machine gates first.** Verification commands from `design.md` run before any reviewer is dispatched; a failing command is a FAIL without spending a review.
2. **Re-reviews are incremental.** Round ≥ 2 checks closure of the previous blocking issues and reviews only the new diff. New blocking issues must come from that diff, or be Critical.
3. **Only blocking issues go back to the implementer.** `[Minor]` and `[Pre-existing]` are recorded, never sent back.
4. **Fix the class.** Blocking issues carry `{class: …}`; fixes and closure checks cover every instance.
5. **Rounds are capped.** 3 consecutive FAILs on one Task, or 3 final-review rounds, stop and ask the user — including the option to revisit the design.
6. **Pre-existing problems never block.** They go to `openspec/backlog.md`.
7. **Approval is bound to a commit.** Phase 6 archives only the HEAD the final review recorded.

## Dependencies

Documented in `README.md`. Pre-flight checks the openspec CLI binary only; plugins are not pre-checked. If a `Skill` invocation fails because a plugin is missing, output:

```
Skill `<name>` not found. This workflow depends on the `superpowers` plugin.
Install via Claude Code plugin manager.
```

then halt.
