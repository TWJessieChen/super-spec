# 03 — Archive 縮成 2–4 個邏輯 commit（取代 squash 成 1 個）

> ⚠ **狀態：未實作。動手前必須先與 repo 擁有者（JC）討論並取得同意。** 討論結果寫進最下方「決議」後才可以改。
> 來源：ultra-spec @ `701e685`（2026-08-22）。比對日期 2026-11-17。

## ultra-spec 的做法（`flows/history-reduction.md`，128 行）

- 實作 commit 數 ≥ 2 時，依模組（切不乾淨時依 Conventional Commit type）分成 2–4 組，最後再加 archive commit。
- **組必須連續**：環境沒有 `git rebase -i`，不重排；區塊交錯時退回「逐 Task 一組」，超過 4 組也接受，並說明原因。
- revision commit 不自成一組，併入其後的組（用 trailer 辨識，依賴 [02](02-commit-anchors.md)）。
- 安全措施：使用者確認分組計畫（硬閘）→ 建 `us-backup-<name>` ref → `reset --hard` + 每組 `read-tree -u --reset` 重放 → `git diff <orig> HEAD` 必須為空（tree 完全相同）。
- planning commit 及以下永不改寫。

## super-spec 現況

`phases/06-archive.md`：所有實作 commit squash 進**一個** archive commit，訊息由 delivered commits 組成。

## 取捨

- 好處：保留交付結構，reviewer 看得出改了哪幾塊，`git bisect` 有落點。
- 代價：唯一會改寫歷史的破壞性流程，需要額外的確認閘與驗證；archive 流程變長。
- super-spec 的「approval 綁定 Reviewed HEAD」：縮減會產生新 hash，要決定 Phase 6 §0 的比對在縮減前做（建議）。

## 當時建議

看個人偏好。若平常 PR 會再 squash merge，價值不大；若直接 push 到主線（目前你的習慣），保留 2–4 個 commit 比較好追。

## 討論時要決定的事

1. 你要一個 commit 還是 2–4 個？要不要做成 archive 時的選項？
2. backup ref 命名與保留多久？
3. 是否先做 [02](02-commit-anchors.md)（分組依賴 trailer）？

## 決議

（尚未討論）
