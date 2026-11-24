# Enforcement ladder — where a recurring mistake gets fixed

**Called from:** `phases/06-archive.md` §4b (recurring finding classes) and `flows/escape.md` §4 (a bug that got past review). Not a standalone flow.

**Why:** a check written as text in `REVIEW.md` only works if a reviewer reads it, notices the diff matches, and complies — every change, at full token cost, and it can still be missed. Agents copy whatever the surrounding code already does, so a mistake that the code base still *allows* keeps coming back. Fix each class at the **strongest level that works**, and keep `REVIEW.md` for what genuinely needs judgment. (Adapted from pstack `/correct` and the "poteto pyramid": codebase → static analysis → verification → prose → human review, cheapest and most reliable first.)

This flow never changes product code. Levels 1–4 are code changes, so they become backlog items that the user later runs as their own `/super-spec` change; level 5 is written now.

## The ladder (strongest first)

| Level | `enforced_by` value | What it means | Example (Android) |
|---|---|---|---|
| 1 structure | `structure:<where>` | The wrong code cannot be written: one owner per piece of state, one supported way per task, internals hidden so the wrong import fails, the old way deleted. | Every `MediaCodec` goes through one `CodecSession` that owns `release()`; direct construction is not visible outside the package. |
| 2 type | `type:<symbol>` | The bad state does not compile. | `@MainThread` / `@WorkerThread` annotations checked by lint; a sealed state type instead of nullable flags. |
| 3 lint / CI | `lint:<rule id>` | A rule fails the build, and **its error message names the fix** (file, type or function to use instead). If the pattern is already common, fail only on additions. | detekt / Android Lint rule: `registerReceiver` outside `ReceiverRegistry` → "use ReceiverRegistry.register(owner, …)". |
| 4 test | `test:<path>` | A regression test that fails when the behavior breaks (and does not fail when only internals change). | Instrumented test that rotates the screen and asserts the player's surface is released. |
| 5 review rule | `reviewer` | A `REVIEW.md` check with a `when:` trigger. Only for things that need judgment or cannot be detected mechanically. | "Every new RTSP retry path has a bounded backoff" — needs reading intent. |

## For each class

1. **Pick the level.** Start at level 1 and go down; stop at the first level that would actually catch every instance of the class. For each level you pass, write one line saying why it does not work here (e.g. `structure: the API is from a third-party SDK we cannot wrap without rewriting 40 call sites`). A class that only ever needs judgment goes straight to 5 — say so.
2. **Repeat check.** If `REVIEW.md` at `RULES_BASE` already has a check for this class (`enforced_by: reviewer`, in `## Always check` or `## Observe`) and the class still recurred, the text rule has failed once already. Level 5 is no longer an option on its own: propose level 1–4 and mark the backlog item `priority: repeat`. (Keep the existing review rule until the stronger one lands.)
3. **Show and ask** (AskUserQuestion, one question per class, up to 4 per call). Options: the proposed level (marked Recommended), the next level down, `review rule only (level 5)`, `skip`. Show the one-line reasons from step 1 in the question text.
4. **Act on the answer:**
   - **Level 1–4** → append a backlog item to `openspec/backlog.md` (create with `# Backlog` if missing):
     ```
     ## <YYYY-MM-DD> — enforce <class> (<level>: <enforced_by value>)
     - [Enforce] <one line: what to build, e.g. "detekt rule ReceiverOutsideRegistry, error names ReceiverRegistry.register"> · class: <class> · priority: <normal|repeat>
     - prove_red_on: <escape case file, or "<commit>: <file:line>" of a real past occurrence from the evidence>
     - then: <what to do with REVIEW.md once it lands — "delete check C7" / "move C7 to ## Enforced elsewhere">
     ```
     `prove_red_on` is required: the check is only trusted once it has failed on a real past mistake (see "Red proof" below). If no real occurrence can be cited, the class does not qualify yet — say so and stop for this class.
     If the class has no review rule yet, **also** add an interim level-5 check (steps below) with `enforced_by: reviewer (pending: <backlog heading>)`, so it is watched until the stronger check lands.
   - **Level 5** → draft the check line and run it through `prompts/rule-skeptic.md` as the calling flow already describes; survivors go under `## Observe` with `· enforced_by: reviewer` appended.
   - **Skip** → record nothing.
5. **Platform lesson (optional).** If the class is about the platform rather than this project (it would recur in any Android / Unity / … project), offer once at the end: "Record <k> of these as platform lessons for future projects?" → Read `flows/lessons.md` §2 (record) for the selected ones.

## Red proof (used when the backlog item is implemented later)

A check never seen to fail proves nothing. When a later `/super-spec` change implements an `[Enforce]` item, Phase 2 writes a `Red proof:` line in `design.md` `## Verification` (see `templates/design.md`), built from `prove_red_on`: a command that runs the new check against the old, broken code and **must fail**. Phase 5 §2c runs it. If the old code passes the new check, the check does not catch the class: `[Critical] {class: check-never-fails}`.

When the change is archived, update `REVIEW.md` as the item's `then:` line says: the review rule is deleted, or moved to `## Enforced elsewhere` (one line: `- <class> → <enforced_by value> (since <change name>)`), so reviewers stop spending tokens on it and the table still shows who enforces what.

## Reply line per class

`<class> (<count>) → <level> <enforced_by> — higher levels: <one-line reasons> — <backlog item | Observe check | skipped>`
