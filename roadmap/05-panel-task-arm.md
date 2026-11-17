# 05 — Panel 門檻刪除「Task 數 ≥ 4」條件

> ⚠ **狀態：未實作。動手前必須先與 repo 擁有者（JC）討論並取得同意。** 討論結果寫進最下方「決議」後才可以改。
> 來源：ultra-spec @ `701e685`（2026-08-22）。比對日期 2026-11-17。

## ultra-spec 的做法

只看 diff 行數（≥ 1500，不含 `openspec/`）；Task 數條件被刪除，理由：
- ultra-spec Phase 2 目標 3–6 個 Task，> 8 是壞味道，Task 數條件形同虛設。
- 更早的版本把 `### Revision N - Task M:` 也算進去，revision 一開始門檻就永遠成立。

## super-spec 現況

`flows/ultracode-review.md` §1：Standard 深度時「Task 數 ≥ 4 **或** diff ≥ 800 行（不含 `openspec/`，2026-11 已改）」。panel 只在 round 1 跑，所以 revision Task 灌水的問題在 super-spec 影響較小；但 recover 後重新 round 1 時，`tasks.md` 會含 revision Task。

## 當時建議

先不改，等累積幾次 `review.md` 的 `## Review stats`（Task 數、diff 行數、panel 是否找到 native 找不到的問題）再決定。也可只改成「不計 revision Task」。

## 討論時要決定的事

1. 刪除 Task 條件、只排除 revision Task，還是維持？
2. 800 行門檻要不要跟著調（ultra-spec 用 1500）？

## 決議

（尚未討論）
