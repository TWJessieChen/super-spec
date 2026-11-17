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

Recommended: a `REVIEW.md` at the repo root (or `openspec/REVIEW.md`) with the project's failure modes, diff-triggered checks, `sensitive_paths`, `quick_paths` and `skip_paths`. Every reviewer reads it — from the commit before the change started, never from the change itself. Pre-flight offers to create one from `templates/REVIEW-rules.md` or the Android example `examples/REVIEW.android.md`. Without it, reviewers use generic rules only and no change is ever classified Deep automatically.

Recommended: a project verify skill at `.claude/skills/verify/` that starts the app and drives its user-facing flows. Final review's Run gate uses it (`design.md` → `Run: skill:verify`). Pre-flight offers to create one from `examples/verify-skill.android/`.

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
│   ├── rule-skeptic.md           # tries to refute a proposed REVIEW.md check before it is added
│   └── archive-committer.md
├── templates/                    # artifact structures
│   ├── proposal.md
│   ├── design.md
│   ├── spec-delta.md             # openspec requirement deltas (merged into living spec at archive)
│   ├── review.md
│   ├── revision-block.md
│   └── REVIEW-rules.md           # project review rules template (copied into the user's repo as REVIEW.md;
│                                 #   not named REVIEW.md so it cannot collide with review.md on case-insensitive filesystems)
├── examples/
│   ├── REVIEW.android.md         # Android / AOSP lifecycle checklist as a REVIEW.md (diff-triggered checks)
│   └── verify-skill.android/     # example project verify skill: adb install → launch → drive → screenshot/logcat
│       └── SKILL.md
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
| Run gate runner (Phase 5 §2b — starts and drives the app) | `sonnet` |
| rule-skeptic (Phase 6 §4b, escape flow) | `sonnet` |

With the **ultracode** engine, the same assignments apply: the table above is auto-derived at HARD-GATE B2, persisted as `## Ultracode Agent Models` in `proposal.md`, and read into the Workflow args — `workflows/apply.js` hardcodes no model names.

## Execution engines (Phase 4)

Chosen per change at Phase 2's **HARD-GATE B2**, persisted as `## Engine` in `proposal.md` (resume never re-asks and never depends on session state):

| Engine | How Phase 4 runs |
|---|---|
| `native` (default) | The orchestrator dispatches each Task's implementer/reviewer one by one — current behavior. |
| `ultracode` | The Task loop runs as one deterministic Workflow script (`workflows/apply.js`): the script splits the work — one fresh implementer + verification run + reviewer agent per Task from `tasks.md` — and severity routing / FAIL counting / model escalation / incremental re-review are code paths. Structured (schema) outputs replace the `Done:`/`PASS`/`FAIL` string contracts; journal-based resume within a session. User gates (Blocked, 3-FAIL cap) surface as workflow pauses routed back to the orchestrator. **Phase 5, round 1**: at Deep depth, or at Standard depth above a size threshold (≥ 4 Tasks or ≥ 800 diff lines, `openspec/` excluded), the final review upgrades to a multi-lens panel (`workflows/final-review.js`): 3 lenses + the holistic reviewer in parallel → 2 skeptics vote on every Critical/Important candidate (downgraded items go to Notes with their original severity, or to Pre-existing) → the VERDICT is computed by the script, a compose-only judge writes the prose (per-role models: derived at HARD-GATE B2, persisted in `proposal.md`'s `## Ultracode Agent Models`). Otherwise Phase 5 stays single-reviewer. Read-only — a dead panel run is simply re-run. |

Ways to enable ultracode: the `--ultra` flag, answering the gate, or asking for it during brainstorming (flips the gate's default). Resume reads `## Engine` from `proposal.md`. Ground rules: git + `tasks.md` remain the only truth source (the workflow journal is cache); the Phase-4 sweep and every history rewrite stay orchestrator-side; Phases 1–3 and 6 are unchanged; Phase 5 uses the multi-lens panel only in round 1 at Deep depth or Standard above the size threshold (otherwise single-reviewer, as with `native`). Details: `flows/ultracode-apply.md`, `flows/ultracode-review.md`.

## Design principles

- **Project-agnostic**: works on any project that has `openspec/` initialized; project-specific review rules (e.g. Android lifecycle checks) live in the project's `REVIEW.md`, not in the prompts
- **No copying of upstream skills**: depends on `superpowers:*` via the Skill tool, so they keep getting upstream updates
- **Strong context isolation**: each subagent dispatch uses the Agent tool with a dedicated prompt template; reviewers see git diffs, never implementer narrative; per-task fresh subagent — see `SKILL.md` `## Context Isolation Rules`
- **Spec-driven, not just design-doc-driven**: Phase 2 writes openspec requirement deltas (`specs/<capability>/spec.md` with scenarios) that serve as acceptance criteria for the final-reviewer and are merged into the living spec by `openspec archive`
- **Reviews converge** (`SKILL.md` `## Review convergence rules`): verification commands run before any reviewer; re-reviews check closure and look only at the new diff; only blocking issues go back to the implementer; fixes cover the whole bug class; round 3+ dispatches no reviewer and hands the open list to the user (HARD-GATE C); a revision that changes code backfills `design.md` / specs in the same commit; pre-existing problems go to the backlog instead of blocking
- **Depth follows risk, not size**: `REVIEW.md` `sensitive_paths` make a change Deep (independent skeptic verification of every blocking finding), docs-only changes are Quick; a change that edits the review rules is always Deep
- **A change cannot modify its own checks**: reviewers read `REVIEW.md` / `CLAUDE.md` from `RULES_BASE` (the commit before the planning commit) via `git show`, never from the working tree
- **Verification means running it**: besides build and tests, final review starts the app and drives the spec scenarios (the Run gate), and records what nobody exercised
- **Checks trigger on the diff**: `REVIEW.md` checks carry a `when:` trigger, so only the 3–10 that match a change are applied; new checks must survive a skeptic and start in `## Observe` (non-blocking) before they can block
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
| 5. Final Review | Full verification command first, then Run (start the app, drive the spec scenarios). Depth (Quick / Standard / Deep) from the touched paths. Round 1: one Opus reviewer (or the ultracode panel) reviews the whole change, walks the spec-delta scenarios, runs `/code-review` and `/security-review` (report-only); Deep adds skeptic verification. Round 2: closure check + revision commits only. Round 3+: HARD-GATE C — no automatic reviewer, the user adjudicates the open list. Writes `review.md` with reviewed HEAD and review stats. | final-reviewer subagent |
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
| Run fails at Phase 5 | `CHANGES REQUESTED` without dispatching a reviewer (`{class: run-failure}`) |
| Run cannot start (no device / emulator) | not the change's fault: recorded as `could not run`, user told which scenarios nobody exercised |
| implementer hits a flaky test | returns `Blocked: flaky test …`; user chooses delete-with-reason / intervene / skip — never skip, sleep, retry or loosen |
| final-reviewer = `CHANGES REQUESTED` (round < 3) | rename `review.md` → `review-r<N>.md`, append `### Revision N - Task M:` to tasks.md, loop back to Phase 4; next round is incremental |
| final review reaches round 3 with issues still open | HARD-GATE C: no reviewer is dispatched; the user picks accept-with-reasons (recorded in `review.md`, archivable) / one more diff round / revisit design / intervene |
| not `APPROVED` after an extra round-3+ review | stop and ask: revisit design / accept remaining issues with reasons / intervene |
| final-reviewer = `NEEDS DESIGN UPDATE` | rename review files to `review-superseded-d<K>-r<N>.md` (round counter resets — next review is a full round 1), append `## Revisions` to design.md, update spec deltas, regenerate tasks.md, loop back to Phase 4 |
| HEAD moved after `APPROVED` | Phase 6 asks: review the new commits / archive with an `Unreviewed:` line / stop |
| a bug shipped that review missed | `flows/escape.md` records it as an eval case and offers REVIEW.md fixes |
| User aborts | choose pause (keep everything) or discard (delete openspec artifacts; code commits left for user to revert manually) |

See `SKILL.md` for full details.

## Verification upgrade (2026-11)

依據 claude.dev〈How the Claude Code team thinks about verification〉（Addy Osmani，2026-11-16）做的改版。目標：驗證層不能被受審的變更修改、要實際把程式跑起來、只在 diff 命中時套用檢查、並用真實漏網 bug 衡量審查品質。下表依優先序列出**已改**項目，最後列出**刻意未改**的項目。

### 已改

| 優先 | 項目 | 改了什麼 | 檔案 |
|---|---|---|---|
| P0 | 規則來自 baseline（變更不能改到審查它的規則） | 所有 reviewer 與 lens 改用 `git show <RULES_BASE>:REVIEW.md` / `CLAUDE.md` 讀規則，`RULES_BASE` = planning commit 的 parent；Phase 5 用同一份快照判斷審查深度；變更有動到 `REVIEW.md`、`openspec/REVIEW.md`、`CLAUDE.md` 時一律 Deep（連 proposal.md 明寫的 Quick／Standard 也會被覆寫），且削弱規則本身是 `[Important] {class: review-rules-weakened}`；`review.md` 新增 `Review rules` 欄；兩支 workflow script 新增 `rulesBase` 參數與 `{RULES_BASE}` placeholder 契約檢查 | `SKILL.md`（Isolation Rule 7）、`prompts/_isolation-preamble.md`、`prompts/task-reviewer.md`、`prompts/final-reviewer.md`、`prompts/final-lens-*.md`、`phases/04-apply.md`、`phases/05-final-review.md`、`workflows/apply.js`、`workflows/final-review.js`、`flows/ultracode-apply.md`、`flows/ultracode-review.md`、`templates/review.md`、`evals/README.md` |
| P1-1 | Run 驗證：實際啟動 app、走 spec 情境 | `design.md` 的 `## Verification` 新增第三行 **Run**（指令或 `skill:<name>`，單元測試不算）；Phase 2 優先採用專案的 verify skill；Phase 5 新增 §2b Run gate：Full 通過後由 sonnet 子代理啟動 app、逐一走 spec-delta 情境、輸出逐步 PASS／FAIL 與證據（存在 `.git/super-spec/<name>/run-evidence/`，不會被 commit）。FAIL → 不派 reviewer、直接 `CHANGES REQUESTED {class: run-failure}`；環境無法啟動（沒有模擬器）→ 記為 `could not run`、不算變更的錯，但要告訴使用者哪些情境沒人跑過。Run 結果併入 `{VERIFICATION}` 給 reviewer 與 spec lens 當證據；沒被 Run 涵蓋的情境標「checked by reading only」。`review.md` 新增 `Run` 欄，archive commit 的 `Testing:` 一併記錄 Run 結果（含 `none`） | `templates/design.md`、`phases/02-propose.md`、`phases/05-final-review.md`、`phases/06-archive.md`、`prompts/final-reviewer.md`、`prompts/final-lens-spec.md`、`workflows/final-review.js`、`templates/review.md`、`SKILL.md`（收斂規則 8） |
| P1-2 | Android verify skill 範本 | 新增 `examples/verify-skill.android/SKILL.md`：adb 確認裝置 → `installDebug` → `am start -W` → 用 uiautomator dump 找元件、`input tap/text` 操作 → screencap 截圖 → `logcat -b crash`／`FATAL EXCEPTION`／`ANR` 健康檢查，附 MediaCodec／RTSP 選用檢查與固定回報格式；pre-flight 新增第 6 節，新變更時若專案沒有 verify skill 會提議從範本建立 | `examples/verify-skill.android/SKILL.md`、`flows/pre-flight.md`、`SKILL.md`（Template index） |
| P2-1 | diff 觸發的檢查清單 | `REVIEW.md` 的 `## Always check` 改為 `- [id] when: <glob／關鍵字> → check: <要看什麼> (source: <案例或 PR>)`；reviewer 只套用 `when:` 命中的條目（沒寫 `when:` 的照舊每次套用，舊格式相容）；Phase 5 記錄 `Checks triggered` 到 `## Review stats`；Android 範例改成 A1–A9 九條帶 trigger 的格式 | `prompts/_isolation-preamble.md`、`templates/REVIEW-rules.md`、`examples/REVIEW.android.md`、`phases/05-final-review.md`、`templates/review.md` |
| P2-2 | 不穩定測試的規定 | implementer 不得 skip／ignore、加 sleep 或拉長 timeout、重跑到通過、放寬斷言；找不到根因就回 `Blocked: flaky test …`。task-reviewer 與 final-reviewer 把這類改動標為 `[Important] {class: flaky-workaround}`。Phase 4 遇到 flaky Blocked 時只給三個選項：寫明原因後刪除測試／手動處理／略過 Task | `prompts/implementer.md`、`prompts/task-reviewer.md`、`prompts/final-reviewer.md`、`phases/04-apply.md`、`SKILL.md`（收斂規則 9） |
| P2-3 | 指定測試類型：property／differential | Phase 3 新增 override 7：重寫、移植、序列化／編解碼、狀態機、遷移、重試／正規化類的 Task 要寫 `Test strategy: property (...)` 或 `differential (...)`；Simple 模式也會寫這些測試；implementer 依語言使用 Kotest／jqwik、Hypothesis、fast-check、proptest；reviewer 找不到對應測試時標 `[Important] {class: missing-test-strategy}` | `phases/03-plan.md`、`prompts/implementer.md`、`prompts/task-reviewer.md`、`prompts/final-reviewer.md` |
| P3-2 | 新規則先觀察再生效 | `REVIEW.md` 新增 `## Observe` 區塊：命中只記 `[Minor] observe:<id>`、永不阻擋、不占 5 條 Minor 上限；Phase 6 與 escape flow 新增的規則一律先放這裡；Phase 6 會列出在 2 份以上 review 有命中的 Observe 規則，問使用者要不要升級到 `## Always check` | `prompts/_isolation-preamble.md`、`templates/REVIEW-rules.md`、`examples/REVIEW.android.md`、`phases/06-archive.md`、`flows/escape.md`、`workflows/apply.js`、`SKILL.md`（收斂規則 10） |
| P3-3 | 新規則要先經過反駁 | 新增 `prompts/rule-skeptic.md`：從「能否從 diff 判斷、trigger 對不對、是否真的重複發生、是否已被覆蓋、是否具體」五點試著推翻提議的規則，可回傳更好的 trigger；Phase 6 §4b 與 escape flow 寫入規則前都要先過這關 | `prompts/rule-skeptic.md`、`phases/06-archive.md`、`flows/escape.md` |
| P3-4 | 分階段量測 recall | `workflows/final-review.js` 回傳 `preVoteCandidates`；native-verify 與 ultracode-review 把投票前的每個候選寫進 `review.md` 新增的 `## Skeptic ledger`；`evals/README.md` 改為分別記錄「投票前抓到」與「投票後還在」，並加入過度擬合的停止條件與保留測試集的做法 | `workflows/final-review.js`、`flows/native-verify.md`、`flows/ultracode-review.md`、`templates/review.md`、`evals/README.md` |

### 刻意未改

| 優先 | 項目 | 為什麼還不改 | 什麼時候做 |
|---|---|---|---|
| P3-1 | 規則定期清理（找出從沒觸發、從沒抓到真 bug 的檢查，提議刪除） | 判斷依據是 archive 後的 `review.md` 裡的 `Checks triggered`、`Observe hits` 與 `## Skeptic ledger`。這些欄位這次才開始記錄，現在沒有任何資料，做了也無法判斷 | 累積約 10 份 archive 後的 `review.md`。預定做法：Phase 6 新增 §4c，彙整最近 10 份紀錄，列出從沒觸發、或觸發過但從未產生 Critical／Important 的條目，詢問後再刪除，不自動刪 |

### 這次也沒有採用的做法

- **commit gate 的 off／observe／enforce 三段模式**：super-spec 不裝 commit hook，archive 前本來就必須通過 final review 且 HEAD 要和審查時一致，等於已經是 enforce；新規則的「先觀察」改用 `## Observe` 實作。
- **加更多 reviewer**：文章指出調好主 reviewer 通常比堆 reviewer 有效；super-spec 只在 Deep 或大型變更才啟用多視角審查，維持不變。
- **用 Stop hook 或 `/goal` 強制跑測試**：orchestrator 本來就在派 reviewer 前自己跑驗證指令，不需要另外加 hook。


## Review convergence upgrade (2026-11, from ultra-spec)

參考 ultra-spec（leepeter121/ultra-spec，super-spec 的 fork）實測數據後移植的三項改動。目標：審查輪次靠設計收斂，而不是靠運氣；panel 門檻量的是程式碼，不是計畫長度。

| 項目 | 改了什麼 | 檔案 |
|---|---|---|
| Panel 門檻排除 `openspec/` | Standard 深度的 diff 行數改用 `git diff --shortstat <baseline>..HEAD -- . ':(exclude)openspec/'`。原本把 `tasks.md`／`design.md` 也算進去，計畫寫得越詳細就越必定觸發 panel（ultra-spec 實測：光 `tasks.md` 1308 行就把 diff 灌到 2180 行） | `flows/ultracode-review.md`、`README.md` |
| Round 3+ 不再自動審查（HARD-GATE C） | Round 1 全審、round 2 diff 模式（原本就是單 reviewer 的 closure + focus）；round ≥ 3 且上一輪是 `CHANGES REQUESTED` 時，只跑 Full／Run 機器關卡，**不派 reviewer**，直接把未關閉清單交給使用者：接受並結束（逐條寫理由，記入 `## Accepted open issues`，可 archive）／再跑一輪 diff review／回頭修設計／手動介入。上一輪是 `APPROVED`（Phase 6 送回來審新 commit）時不觸發。設計修訂（`flows/recover.md`）會把 review 檔改名成 `review-superseded-d<K>-r<N>.md` 移出計數，下一輪重新從 round 1 全審——否則從 gate C 進 recover 後，回來會直接又撞上 gate C | `phases/05-final-review.md`（§4、新增 §4b、§7）、`flows/recover.md`、`flows/resume-detection.md`、`flows/todo-tracking.md`、`SKILL.md`（收斂規則 5、flow index） |
| Revision 動到程式碼就同一個 commit 回填 design／spec | Phase 5 新增「Revision task rules」：`**Files:**` 有 `openspec/**` 以外路徑的 revision task，最後一個 sub-step 必須回填 `design.md` 與受影響的 spec，且這些文件要列進 `**Files:**`；implementer 新增 *Design-truth backfill* 節（只修事實描述；要改設計決策就回 `Blocked`）；task-reviewer 新增 design-truth sync 檢查，未同步為 `[Important] {class: design-drift}`。依據：ultra-spec 第三輪留下的 `[Important]` 有 3/4 是上一輪 revision 造成的文件與程式碼不一致 | `phases/05-final-review.md`、`flows/recover.md`、`prompts/implementer.md`、`prompts/task-reviewer.md` |

沒有移植的：brainstorm／propose 的多 agent 擴散、cost-fit 模型分配、waves 平行 apply、只剩 ultracode 引擎（會失去 native 退路），以及 archive 縮成 2–4 個 commit。
