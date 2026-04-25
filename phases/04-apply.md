# Phase 4 — Apply

Loop over each Task in `tasks.md` that is not yet complete (`- [ ]` at the Task header level).

For each Task, do steps 1 → 4 in order.

## 1. Dispatch the implementer subagent

Use the **Agent** tool (not Skill — we need fresh-context isolation).

Read `prompts/implementer.md` (relative to this skill's directory) and substitute the variables. Pass the resulting text as the agent's prompt.

Variables:
- `{TASK_NUMBER}`: Task index (e.g., 3)
- `{TASK_BODY}`: This Task's section from tasks.md only (header + sub-steps)
- `{DESIGN_PATH}`: `openspec/changes/<name>/design.md`
- `{MODE}`: `TDD` or `Simple`
- `{RELEVANT_FILES}`: list of existing files this Task touches (extract from the Task body's `**Files:**` section)

**Model:** inherit (omit the `model` parameter on the Agent call).

**Forbidden in this prompt** (do not include):
- Brainstorming conversation
- `proposal.md` content (apart from the Mode flag, which is passed via `{MODE}`)
- Other Tasks' content
- Other Tasks' reviewer reports
- Any other implementer's response or narrative

The implementer is contractually required to return one line: `Done: <commit_hash>` or `Blocked: <reason>`.

## 2. Dispatch the spec-reviewer subagent

Use the **Agent** tool. Read `prompts/spec-reviewer.md` and substitute:
- `{COMMIT_HASH}`: the hash from the implementer's response
- `{DESIGN_PATH}`: `openspec/changes/<name>/design.md`
- `{TASK_BODY}`: the same Task section

**Model:** `claude-sonnet-4-6`

**Forbidden:** the implementer's narrative; other Tasks' content; quality-reviewer outputs.

The reviewer returns `PASS` or `FAIL\n- <issues>`.

If `FAIL`, re-dispatch the implementer (step 1) with the FAIL issues appended to the prompt under a `## Previous review failed with these issues` section. Track the failure count.

After the **3rd consecutive FAIL** on the same Task, **pause** the workflow:
- Output the latest reviewer report to the user
- Ask: "Reviewer has failed 3 times. How would you like to proceed? (e.g., adjust the design, skip this task, manually intervene)"
- Wait for user direction

## 3. Dispatch the quality-reviewer subagent

Use the **Agent** tool. Read `prompts/quality-reviewer.md` and substitute:
- `{COMMIT_HASH}`
- `{TASK_BODY}` (for context only)

**Model:** `claude-sonnet-4-6`

**Forbidden:** `design.md` (quality should not depend on design intent); the implementer's narrative; the spec-reviewer report.

Same FAIL / retry / pause-after-3 behavior as spec-reviewer.

## 4. Mark Task complete

Use the Edit tool to flip `- [ ]` → `- [x]` for:
- Each sub-step under this Task (if not already done by the implementer)
- The Task header itself

## 5. Loop

If more Tasks remain with `- [ ]`, return to step 1 with the next Task. Otherwise → Phase 5.
