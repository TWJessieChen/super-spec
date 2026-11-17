# roadmap/ — 未來可能的改動（尚未實作）

> ⚠ **任何一項要動手前，都必須先與 repo 擁有者（JC）討論，確認要做、怎麼做之後才實作。**
> 這個資料夾只記錄「候選改動」與當時的分析，**不是已核准的計畫**。不可以因為讀到這裡就直接改 skill。

## 這個資料夾是什麼

2026-11 比對 ultra-spec（[leepeter121/ultra-spec](https://github.com/leepeter121/ultra-spec)，super-spec 的 fork，比對時的 commit `701e685`）後，已移植三項（panel 門檻排除 `openspec/`、round 3+ HARD-GATE C、revision 回填 design/spec，見根目錄 `README.md`「Review convergence upgrade」）。其餘沒移植的項目寫在這裡，每項一個檔案，內容包含：

- ultra-spec 怎麼做、來源檔位置
- 移植到 super-spec 要動哪些檔
- 取捨與風險
- 當時的建議
- **討論時需要決定的問題**

## 規則

1. **執行 `/super-spec` 時不讀這個資料夾。** 它不是 phase/flow，`SKILL.md` 不索引它，不是執行輸入。
2. **要實作某一項：先開對話跟擁有者討論**，帶著該檔的「討論時要決定的事」逐條確認；討論結果寫回該檔的「決議」欄後才開始改。
3. 實作完成後：把該檔移到 `roadmap/done/`（或刪除），並在根目錄 `README.md` 補一節說明改了什麼。
4. 決定不做的項目：保留檔案，在「決議」欄寫「不做」與理由，避免日後重複評估。
5. ultra-spec 的來源檔可能已更新；動手前重新比對來源，不要只信這裡的摘要。

## 清單

| # | 項目 | 當時建議 | 成本／風險 | 檔案 |
|---|---|---|---|---|
| 1 | Final-review 改成 core lens + adversarial + over-engineering 三 lens | 推薦優先討論 | 中（prompt + final-review.js） | [01-attack-lenses.md](01-attack-lenses.md) |
| 2 | Commit 定位改用路徑／trailer，不靠 subject 字樣 | 推薦（可攜性 bug） | 低 | [02-commit-anchors.md](02-commit-anchors.md) |
| 3 | Archive 縮成 2–4 個邏輯 commit（取代 squash 成 1 個） | 看偏好 | 中，有改寫歷史的風險 | [03-history-reduction.md](03-history-reduction.md) |
| 4 | `tasks.md` lint 腳本（禁止貼完整實作） | 可考慮 | 低，但多一個 Python 依賴 | [04-tasks-lint.md](04-tasks-lint.md) |
| 5 | Panel 門檻刪除「Task 數 ≥ 4」條件 | 等資料再說 | 低 | [05-panel-task-arm.md](05-panel-task-arm.md) |
| 6 | Propose 階段 critic 一次審四份文件 | 可考慮 | 中，每個變更多一次 dispatch | [06-propose-critic.md](06-propose-critic.md) |
| 7 | Plan-review gate + waves 平行 apply（worktree） | 暫不建議 | 高 | [07-waves-parallel-apply.md](07-waves-parallel-apply.md) |
| 8 | Cost-fit 模型分配 + 每次啟動前確認 | 暫不建議 | 中 | [08-cost-fit-models.md](08-cost-fit-models.md) |
| 9 | Brainstorm 設計空間多 agent 擴散（explorers + judge） | 暫不建議 | 中高（每個變更都付成本） | [09-brainstorm-fanout.md](09-brainstorm-fanout.md) |
| 10 | 只保留 ultracode 引擎、移除 native | 不建議 | 高，失去退路 | [10-ultracode-only.md](10-ultracode-only.md) |
