# super-spec

Single-entry slash command that orchestrates a non-trivial change end-to-end through:

```
brainstorm → propose → plan → apply (per-task subagent + verification gate + review) → final review (rounds, depth by risk) → archive
```

All artifacts live in `openspec/changes/<name>/`. Defaults: TDD mode is offered, no merge / push / PR.

## Trigger

```
/super-spec [--ultra] [<name | short description>]
```

- No argument → asks what you want to build
- New `<name>` → starts at brainstorm
- Existing `<name>` → resumes from current state
- `--ultra` → pre-answers the Phase-2 engine gate with `ultracode` (see "Execution engines" below)

## Dependencies

Required on every machine that runs this skill:

| Dependency | Install |
|---|---|
| `openspec` CLI | `npm i -g @openspec/cli` (see https://openspec.dev) |
| `superpowers` plugin | Claude Code plugin manager — provides `superpowers:brainstorming`, `superpowers:writing-plans`, etc. |
| `code-review` skill | Built-in to Claude Code — no install needed |
| `security-review` command | Built-in to Claude Code — used by the final review at Standard / Deep depth; the reviewer does the analysis itself if it is unavailable |

The skill checks `openspec --version` at startup and halts with install instructions if missing. Plugin presence is **not** pre-checked; a missing plugin surfaces as a clear error when its skill is first invoked.

## Project requirements

The current project must have `openspec/` initialized. If not, the skill will offer to run `openspec init`.

The skill works on whatever git branch is currently checked out — it does **not** create a new branch. All commits go on the current branch.

The skill refuses to start if there are tracked-file changes (modified or staged). Untracked files are allowed and do not block startup.

Recommended: a `REVIEW.md` at the repo root (or `openspec/REVIEW.md`) with the project's failure modes, `sensitive_paths`, `quick_paths` and `skip_paths`. Every reviewer reads it. Pre-flight offers to create one from `templates/REVIEW.md` or the Android example `examples/REVIEW.android.md`. Without it, reviewers use generic rules only and no change is ever classified Deep automatically.

## Layout

```
super-spec/
├── SKILL.md                      # entry point — an INDEX ONLY; logic lives in the sibling files below
├── phases/
│   ├── 01-brainstorm.md
│   ├── 02-propose.md
│   ├── 03-plan.md
│   ├── 04-apply.md
│   ├── 05-final-review.md
│   └── 06-archive.md
├── flows/                        # non-linear paths
│   ├── pre-flight.md
│   ├── resume-detection.md
│   ├── phase4-sweep.md           # Phase-4 entry sweep (superpowers residue purge, both engines)
│   ├── recover.md                # design revision loop (NEEDS DESIGN UPDATE, or "revisit design" at a round cap)
│   ├── todo-tracking.md          # visualized progress list
│   ├── ultracode-apply.md        # Phase 4 via Workflow engine (orchestrator side)
│   ├── ultracode-review.md       # Phase 5 multi-lens panel (orchestrator side)
│   ├── native-verify.md          # Phase 5 skeptic voting without the Workflow engine (Deep depth)
│   ├── escape.md                 # record a bug that got past review as an eval case
│   └── abort.md
├── workflows/
│   ├── apply.js                  # static Workflow script for the ultracode Phase-4 Task loop
│   └── final-review.js           # static Workflow script for the Phase-5 panel
├── prompts/                      # subagent prompt templates
│   ├── _isolation-preamble.md    # shared by every prompt below
│   ├── implementer.md
│   ├── task-reviewer.md          # single per-task reviewer: spec compliance + code quality
│   ├── final-reviewer.md         # sole reviewer (native / below threshold); holistic lens (panel)
│   ├── final-lens-spec.md        # panel lens: spec-scenario walking
│   ├── final-lens-lifecycle.md   # panel lens: resource/lifecycle/threading
│   ├── final-lens-coherence.md   # panel lens: cross-task DRY / interface / gaps
│   ├── final-skeptic.md          # panel: adversarial verify of one gating issue
│   ├── final-judge.md            # panel: compose-only (verdict is read-only input)
│   └── archive-committer.md
├── templates/                    # artifact structures
│   ├── proposal.md
│   ├── design.md
│   ├── spec-delta.md             # openspec requirement deltas (merged into living spec at archive)
│   ├── review.md
│   ├── revision-block.md
│   └── REVIEW.md                 # project review rules template (copied into the user's repo)
├── examples/
│   └── REVIEW.android.md         # Android / AOSP lifecycle checklist as a REVIEW.md
├── evals/
│   └── README.md                 # how to re-score reviewer prompt changes against recorded escapes
└── README.md                     # this file
```

`SKILL.md` is an index: each phase/flow/template file is Read at the moment it is entered, never paraphrased from memory.

## Subagent model assignment

Model parameters use Agent-tool **aliases** (`sonnet` / `opus` / `haiku`), not pinned model IDs — aliases track the latest version automatically.

| Subagent | Model |
|---|---|
| implementer (TDD (Sonnet) mode) | `sonnet` |
| implementer (TDD (Opus) mode) | `opus` |
| implementer (Simple mode) | inherit (parent's model) |
| implementer (3rd dispatch after 2 consecutive FAILs) | escalated to `opus` regardless of mode |
| task-reviewer | `sonnet` |
| verification command runner / git-ops | `haiku` |
| final-reviewer | `opus` (`sonnet` at Quick depth) |
| final-skeptic (panel, or native-verify at Deep depth) | `sonnet` |
| archive-committer | `haiku` |

With the **ultracode** engine, the same assignments apply: the table above is auto-derived at HARD-GATE B2, persisted as `## Ultracode Agent Models` in `proposal.md`, and read into the Workflow args — `workflows/apply.js` hardcodes no model names.

## Execution engines (Phase 4)

Chosen per change at Phase 2's **HARD-GATE B2**, persisted as `## Engine` in `proposal.md` (resume never re-asks and never depends on session state):

| Engine | How Phase 4 runs |
|---|---|
| `native` (default) | The orchestrator dispatches each Task's implementer/reviewer one by one — current behavior. |
| `ultracode` | The Task loop runs as one deterministic Workflow script (`workflows/apply.js`): the script splits the work — one fresh implementer + verification run + reviewer agent per Task from `tasks.md` — and severity routing / FAIL counting / model escalation / incremental re-review are code paths. Structured (schema) outputs replace the `Done:`/`PASS`/`FAIL` string contracts; journal-based resume within a session. User gates (Blocked, 3-FAIL cap) surface as workflow pauses routed back to the orchestrator. **Phase 5, round 1**: at Deep depth, or at Standard depth above a size threshold (≥ 4 Tasks or ≥ 800 diff lines), the final review upgrades to a multi-lens panel (`workflows/final-review.js`): 3 lenses + the holistic reviewer in parallel → 2 skeptics vote on every Critical/Important candidate (downgraded items go to Notes with their original severity, or to Pre-existing) → the VERDICT is computed by the script, a compose-only judge writes the prose (per-role models: derived at HARD-GATE B2, persisted in `proposal.md`'s `## Ultracode Agent Models`). Otherwise Phase 5 stays single-reviewer. Read-only — a dead panel run is simply re-run. |

Ways to enable ultracode: the `--ultra` flag, answering the gate, or asking for it during brainstorming (flips the gate's default). Resume reads `## Engine` from `proposal.md`. Ground rules: git + `tasks.md` remain the only truth source (the workflow journal is cache); the Phase-4 sweep and every history rewrite stay orchestrator-side; Phases 1–3 and 6 are unchanged; Phase 5 uses the multi-lens panel only in round 1 at Deep depth or Standard above the size threshold (otherwise single-reviewer, as with `native`). Details: `flows/ultracode-apply.md`, `flows/ultracode-review.md`.

## Design principles

- **Project-agnostic**: works on any project that has `openspec/` initialized; project-specific review rules (e.g. Android lifecycle checks) live in the project's `REVIEW.md`, not in the prompts
- **No copying of upstream skills**: depends on `superpowers:*` via the Skill tool, so they keep getting upstream updates
- **Strong context isolation**: each subagent dispatch uses the Agent tool with a dedicated prompt template; reviewers see git diffs, never implementer narrative; per-task fresh subagent — see `SKILL.md` `## Context Isolation Rules`
- **Spec-driven, not just design-doc-driven**: Phase 2 writes openspec requirement deltas (`specs/<capability>/spec.md` with scenarios) that serve as acceptance criteria for the final-reviewer and are merged into the living spec by `openspec archive`
- **Reviews converge** (`SKILL.md` `## Review convergence rules`): verification commands run before any reviewer; re-reviews check closure and look only at the new diff; only blocking issues go back to the implementer; fixes cover the whole bug class; 3 rounds then ask; pre-existing problems go to the backlog instead of blocking
- **Depth follows risk, not size**: `REVIEW.md` `sensitive_paths` make a change Deep (independent skeptic verification of every blocking finding), docs-only changes are Quick
- **Approval is bound to a commit**: `review.md` records the reviewed HEAD; archive refuses silently-changed history
- **Measured**: every `review.md` carries `## Review stats` and finding dispositions; escaped bugs become eval cases (`flows/escape.md`, `evals/README.md`)
- **Conservative defaults**: no auto-merge, no auto-push, no PR; `archive` only on explicit user instruction after `APPROVED`
- **Honest abort**: `discard` deletes openspec artifacts but reports (not reverts) any code commits accumulated during the change

## Quick reference: the six phases

| Phase | What happens | Who writes |
|---|---|---|
| 1. Brainstorm | Pure dialogue via `superpowers:brainstorming`. No files written. | (none) |
| 2. Propose | HARD-GATE: TDD (Sonnet) / TDD (Opus) / Simple? Engine? Then `openspec new change`, write `proposal.md` (incl. `## Review Depth`) + `design.md` (incl. `## Verification` commands) + `specs/<capability>/spec.md` deltas. | orchestrator |
| 3. Plan | Invoke `superpowers:writing-plans`, write `tasks.md` with TDD or Simple sub-steps. One `openspec(<name>): planning` commit covers all planning artifacts. | orchestrator |
| 4. Apply | Loop per Task: implementer → per-task verification command (machine gate) → task-reviewer (spec compliance + quality, severity-tagged, bug class; round ≥ 2 reviews only the interdiff). Each is a fresh subagent. TDD Tasks leave red-phase evidence as a separate `test(...)` commit. Review rounds are recorded in `tasks.md`. | implementer subagent |
| 5. Final Review | Full verification command first. Depth (Quick / Standard / Deep) from the touched paths. Round 1: one Opus reviewer (or the ultracode panel) reviews the whole change, walks the spec-delta scenarios, runs `/code-review` and `/security-review` (report-only); Deep adds skeptic verification. Round ≥ 2: closure check + revision commits only. Writes `review.md` with reviewed HEAD and review stats. | final-reviewer subagent |
| 6. Archive | Only on explicit user instruction after `APPROVED`, and only if HEAD is still the reviewed HEAD. Pre-existing items go to `openspec/backlog.md`; `openspec archive` merges spec deltas into `openspec/specs/`; Haiku subagent commits (message includes `Testing:` / `Risk:`). Offers to add recurring finding classes to `REVIEW.md`. STOP — no merge / push / PR. | archive-committer subagent |

## Recovery paths

| Situation | Handling |
|---|---|
| implementer returns `Blocked: <reason>` | pause, surface the reason, ask the user how to proceed |
| task-reviewer FAILs 2× consecutively on same Task | 3rd implementer dispatch escalates to `opus` |
| verification command fails after a Task | counts as a FAIL; implementer re-dispatched with the output tail, no reviewer dispatched |
| task-reviewer FAILs 3× on same Task | pause, surface latest report; options: revisit design / rewrite Task / skip / intervene |
| task-reviewer output malformed | re-dispatch reviewer once with a note; still malformed → pause and surface |
| full verification fails at Phase 5 | `CHANGES REQUESTED` without dispatching a reviewer |
| final-reviewer = `CHANGES REQUESTED` (round < 3) | rename `review.md` → `review-r<N>.md`, append `### Revision N - Task M:` to tasks.md, loop back to Phase 4; next round is incremental |
| not `APPROVED` after round 3 | stop and ask: revisit design / accept remaining issues with reasons / intervene |
| final-reviewer = `NEEDS DESIGN UPDATE` | append `## Revisions` to design.md, update spec deltas, regenerate tasks.md, loop back to Phase 4 |
| HEAD moved after `APPROVED` | Phase 6 asks: review the new commits / archive with an `Unreviewed:` line / stop |
| a bug shipped that review missed | `flows/escape.md` records it as an eval case and offers REVIEW.md fixes |
| User aborts | choose pause (keep everything) or discard (delete openspec artifacts; code commits left for user to revert manually) |

See `SKILL.md` for full details.
