# 02 — Commit 定位改用路徑／trailer，不靠 subject 字樣

> ⚠ **狀態：未實作。動手前必須先與 repo 擁有者（JC）討論並取得同意。** 討論結果寫進最下方「決議」後才可以改。
> 來源：ultra-spec @ `701e685`（2026-08-22）。比對日期 2026-11-17。

## 問題

super-spec 用 commit subject 找 planning commit：`git log --grep="openspec(<name>): planning" -n 1`（`phases/04-apply.md`、`phases/05-final-review.md` §1、`phases/06-archive.md` 兩處、`flows/phase4-sweep.md`）。

- 若專案規定 commit subject 用中文或其他格式（例：`openspec(<name>): 規劃文件`），`--grep` 會**靜默回傳空值**，baseline、`RULES_BASE`、`COMMIT_RANGE` 全部算錯。
- `git log --grep … -n 1` 拿到的是**最新**符合的那個，不是最早的。

## ultra-spec 的做法（`flows/commit-anchors.md`，35 行）

1. **Planning commit 用路徑找**：第一個新增 `openspec/changes/<name>/` 的 commit
   `git log --diff-filter=A --reverse --format='%H' -- "openspec/changes/<name>/" | head -1`
   （必須 `--reverse | head -1`，**不可**寫 `--reverse -n 1`——`-n` 在 reverse 前套用，會拿到錯的 commit）。
2. **Revision 輪次用 trailer 找**：revision commit body 帶 `Ultra-Spec: <name> revision <N>` trailer。
3. 已知限制：同名 change 被 discard 後重用，會有兩次新增事件；交給 abort 流程的「planning commit 不在歷史裡 → 問使用者」處理。

## 移植要動的檔

新增 `flows/commit-anchors.md`；改 `phases/04-apply.md`、`phases/05-final-review.md`、`phases/06-archive.md`、`flows/phase4-sweep.md`、`flows/abort.md`、`flows/recover.md`（revision commit 加 trailer）。

## 當時建議

推薦。改動小、純粹修可攜性 bug，不改變行為。

## 討論時要決定的事

1. trailer 名稱用什麼（例如 `Super-Spec: <name> revision <N>`）？
2. 舊的、沒有 trailer 的進行中變更要不要相容（fallback 回 subject 搜尋）？

## 決議

（尚未討論）
